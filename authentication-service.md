# Authentication Service

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



