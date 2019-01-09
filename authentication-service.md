# Authentication Service


## Adding a Custom Rule - User Role

For our API, one of the endpoints also requires a custom claim. Our Angular application, or Token Minter thus far, does not have permission alone to grant this scope. We will implement a custom Auth0 Rule, that will allow us to add this custom roles claim to our access_token.

Auth0 Rules are JavaScript functions that are executed in Auth0 as part of the transaction every time a user authenticates. In this way, rules allow you to easily customize and extend your authentication process. We want to use a rule to add a role to users that log into our app. In this manner, we can authorize specific user roles in different ways.

Go to the Rules section in the Dashboard sidebar and click the +Create Rule button. Choose the Empty Rule template. (Alternatively, you could choose the Set roles to a user template and modify it according to your needs.)

Add the following code as shown below:

```js
// Set roles to a user
function (user, context, callback) {
  // Make sure the user has verified their email address
  if (!user.email || !user.email_verified) {
    return callback(new UnauthorizedError('Please verify your email before logging in.'));
  }
  user.app_metadata = user.app_metadata || {};
  var addRolesToUser = function(user, cb) {
    // Replace {YOUR_FULL_EMAIL_HERE} with your own email address
    if (user.email && user.email === '{YOUR_FULL_EMAIL_HERE}') {
      cb(null, ['editor']);
    } else {
      cb(null, []);
    }
  };
  addRolesToUser(user, function(err, roles) {
    if (err) {
      callback(err);
    } else {
      user.app_metadata.roles = roles;
      auth0.users.updateAppMetadata(user.user_id, user.app_metadata)
        .then(function(){
          var namespace = 'https://secure-dino-api/roles';
          var userRoles = user.app_metadata.roles;
          context.idToken[namespace] = userRoles;
          context.accessToken[namespace] = userRoles;
          callback(null, user, context);
        })
        .catch(function(err){
          callback(err);
        });
    }
  });
}
```
Set up a pattern for the editor user to be specifically identified. The example above uses email matching with strict equality. Make sure you also add the app metadata containing your roles to the accessToken as well as the idToken. The access token will provide data to our API so we can verify that the user has the appropriate role when they request resources.

Note: You can use any type of condition you'd like to identify admin users: by email, provider, name, domain, etc.

The namespace identifier in the addRolesToUser() callback function can be any non-Auth0 HTTP or HTTPS URL and does not have to point to an actual resource. Auth0 enforces this recommendation from OIDC regarding additional claims and will silently exclude any claims that do not have a namespace. You can read more about implementing custom claims with Auth0 here.

Click the Save button to save your rule. It will then be enabled by default.


## Configuring Auth Service

Open up the `environments.ts` file and add the following configuration:

```js
export const environment = {
  production: false,
  auth: {
    clientId: '{YOUR_AUTH0_CLIENT_ID}',
    domain: '{YOUR_AUTH0_DOMAIN}', // e.g., you.auth0.com
    redirect: 'http://localhost:4200/callback',
    logoutUrl: 'http://localhost:4200',
    roles_namespace: '{YOUR_ROLES_NAMESPACE}' // e.g., https://secure-dino-api/roles
  },
  api: {
    baseUrl: '{YOUR_API_URL}' // e.g., http://localhost:3005/api/
  }
};
```

## Building the Auth Service

The Authentication service supplies our app with the methods it needs to log in, log out, set up user sessions, and automatically renew tokens. Open `auth.service.ts` and add the following code:

```js
// src/app/auth/auth.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, bindNodeCallback } from 'rxjs';
import * as auth0 from 'auth0-js';
import { environment } from './../../environments/environment';
import { Location } from '@angular/common';
import { Router } from '@angular/router';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  // Create Auth0 web auth instance
  // @TODO: Update environment variables and remove .sample
  // extension in src/environments/environment.ts.sample
  // and src/environments/environment.prod.ts.sample
  private Auth0 = new auth0.WebAuth({
    clientID: environment.auth.clientId,
    domain: environment.auth.domain,
    responseType: 'token id_token',
    audience: 'https://secure-dino-api',
    redirectUri: environment.auth.redirect,
    scope: 'openid profile email read:dino-details write:dino-fav'
  });
  // localStorage property names
  private authFlag = 'isLoggedIn';
  // Store access token and create stream
  accessToken: string = null;
  accessToken$ = new BehaviorSubject<string>(this.accessToken);
  // Create stream of user profile data
  userProfile: any = null;
  userProfile$ = new BehaviorSubject<any>(this.userProfile);
  // Auth-related URL paths
  logoutPath = '/';
  defaultSuccessPath = '/';
  // Create observable of Auth0 parseHash method; gather auth results
  parseHash$ = bindNodeCallback(this.Auth0.parseHash.bind(this.Auth0));
  // Create observable of Auth0 checkSession method to
  // verify authorization server session and renew tokens
  checkSession$ = bindNodeCallback(this.Auth0.checkSession.bind(this.Auth0));

  // Token expiration management
  accessTokenExp: number;
  // Hide auth header while performing local login
  // (e.g., on the callback page)
  hideAuthHeader: boolean;

  constructor(
    private router: Router,
    private location: Location
  ) { }

  login() {
    this.Auth0.authorize();
  }

  handleLoginCallback() {
    if (window.location.hash && !this.isAuthenticated) {
      // Hide header while parsing hash
      this.hideAuthHeader = true;
      // Subscribe to parseHash$ bound callback observable
      this.parseHash$({}).subscribe(
        authResult => {
          // Don't keep callback route + hash in browser history
          this.location.replaceState('/');
          // Log in locally and navigate
          this.localLogin(authResult);
          this.router.navigateByUrl(this.defaultSuccessPath);
        },
        err => this.handleError(err)
      );
    } else {
      // If visiting the callback page with no hash
      // return to default logged out route
      this.goToLogoutUrl();
    }
  }

  renewAuth() {
    if (this.isAuthenticated) {
      // Check Auth0 authorization server session
      this.checkSession$({}).subscribe(
        authResult => this.localLogin(authResult),
        err => this.handleError(err)
      );
    }
  }

  private localLogin(authResult) {
    if (authResult && authResult.accessToken && authResult.idToken && authResult.idTokenPayload) {
      // Set token expiration
      const now = new Date().getTime();
      this.accessTokenExp = now + (authResult.expiresIn * 1000);
      // Set token in local property and emit in stream
      this.setToken(authResult.accessToken);
      // Emit value for user profile stream
      this.userProfile = authResult.idTokenPayload;
      this.userProfile$.next(this.userProfile);
      // Set flag in local storage stating app is logged in
      localStorage.setItem(this.authFlag, JSON.stringify(true));
    } else {
      // Something was missing from expected authResult
      this.localLogout();
    }
    this.hideAuthHeader = false;
  }

  private localLogout() {
    this.userProfile$.next(null);
    this.setToken(null);
    localStorage.setItem(this.authFlag, JSON.stringify(false));
    this.goToLogoutUrl();
  }

  logout() {
    this.localLogout();
    // Auth0 server logout does a full page redirect:
    // make sure you have full logout URL in your Auth0
    // Dashboard Application settings in Allowed Logout URLs
    this.Auth0.logout({
      returnTo: environment.auth.logoutUrl,
      clientID: environment.auth.clientId
    });
  }

  private handleError(err) {
    this.hideAuthHeader = false;
    console.error(err);
    // Log out locally and redirect to default auth failure route
    this.localLogout();
  }

  get isAuthenticated(): boolean {
    // Check if the Angular app thinks this user is authenticated
    return JSON.parse(localStorage.getItem(this.authFlag));
  }

  setToken(token: string) {
    this.accessToken = token;
    this.accessToken$.next(token);
  }

  goToLogoutUrl() {
    this.router.navigate([this.logoutPath]);
  }

  userHasRole(reqRole: string): boolean {
    return this.userProfile[environment.auth.roles_namespace].indexOf(reqRole) > -1;
  }

}
```

We will use Auth0's `/authorize` endpoint \(called with the [auth0.js](https://auth0.com/docs/libraries/auth0js) method `authorize()`\) to open the [Auth0 login page](https://auth0.com/docs/hosted-pages/login) and send users to a [centralized authorization server for authentication](https://auth0.com/blog/authentication-provider-best-practices-centralized-login/). Upon successful authentication, the authorization server will then redirect the user back to our application with a URL hash, which can be parsed to extract auth results. The user's access token, token expiration, profile, and desired redirect URL \(if they were trying to access a guarded route\) are saved in memory. If the user's access token expires while they are still using the app, their session will be automatically renewed.

> **Note**: If the user logged in with a [social identity provider](/auth0-setup.md#set-up-social-identity-providers) and Auth0 dev keys are set up for the connection, any attempts to renew the session silently will return a `login_required` error. To avoid this error, set up client accounts with all social IdPs.

## Update Root Component

Next, go into your `app.component.ts` file and in the `ngOnInit` method add the code to renew user authentication whenever the application is reloaded.

```js
import { Component, OnInit } from '@angular/core';
import { AuthService } from './auth/auth.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {

  constructor(public auth: AuthService) {}

  ngOnInit() {
    // If the app believes there is an active session
    // on Auth0 authorization server, attempt to renew auth
    this.auth.renewAuth();
  }
}
```

Renew user authentication when the app is reloaded. The `ngOnInit` lifecycle hook runs whenever the App component is initiated.



