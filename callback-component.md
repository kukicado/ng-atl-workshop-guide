# Callback Component

The Callback component is necessary to enable authentication. In our [Authentication service](//authentication-module.md#authentication-service), we created a method to parse the authentication hash, but we haven't actually _called_ this method from anywhere. The appropriate place for this method is in the [Callback page, which is where the user is redirected after authenticating with Auth0](/auth0-setup.md#create-a-client).

Open the `callback.component.ts` file and add:

```js
// src/app/pages/callback.component.ts
import { Component, OnInit } from '@angular/core';
import { Title } from '@angular/platform-browser';
import { AuthService } from '../auth/auth.service';

@Component({
  selector: 'app-callback',
  template: `
    <app-loading></app-loading>
  `,
  styles: []
})
export class CallbackComponent implements OnInit {

  constructor(
    private title: Title,
    private auth: AuthService
  ) { }

  ngOnInit() {
    this.title.setTitle('Angular Authentication with Auth0');
    this.auth.handleLoginCallback();
  }

}
```

After login, the user will be redirected back to our app with a URL that looks something like this:

```
http://localhost:4200/callback#{authentication_result_hash_with_jwt}
```

The `auth.handleLoginCallback()` method parses the auth result hash, requests the user's info, sets up the session, and then redirects the user to the appropriate route in the app.

