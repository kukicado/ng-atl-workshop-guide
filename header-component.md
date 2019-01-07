# Header Component

Now that we have authentication, route guards, and a way to handle the authentication hash set up, we need a way for the user to log into our app! We'll do this in the Header component.

## Header Component Class

Open the `auth-header.component.ts` class file:

```js
// src/app/auth/auth-header/auth-header.component.ts
import { Component, OnInit } from '@angular/core';
import { AuthService } from '../auth.service';

@Component({
  selector: 'app-auth-header',
  templateUrl: './auth-header.component.html',
  styles: [`
    img {
      border-radius: 100px;
      height: 30px;
      width: 30px;
    }
    .active { font-weight: bold; }
  `]
})
export class AuthHeaderComponent implements OnInit {

  constructor(public auth: AuthService) { }

  ngOnInit() {
  }

}
```

We'll provide some CSS styles and make the Authentication service publicly available in the constructor function so its methods can be used in the template.

## Header Component Template

Now open the `auth-header.component.html` template file and add:

```html
<!-- src/app/auth/auth-header/auth-header.component.html -->
<nav *ngIf="!auth.hideAuthHeader" class="nav justify-content-between mt-2 mx-2 mb-3">
  <ul class="nav">
    <li class="nav-item">
      <a
        routerLink="/"
        class="nav-link"
        routerLinkActive="active"
        [routerLinkActiveOptions]="{ exact: true }">Home</a>
    </li>
    <li class="nav-item">
      <a
        routerLink="/dinosaurs"
        class="nav-link"
        routerLinkActive="active">Dinosaurs</a>
    </li>
  </ul>

  <div id="authContainer" class="ml-3">
    <button
      *ngIf="!auth.isAuthenticated"
      class="btn btn-success btn-sm"
      (click)="auth.login()">Log In</button>
    <span *ngIf="(auth.userProfile$ | async) as user">
      <img [src]="user.picture">
      <a class="px-1" routerLink="/profile" href><small>{{ user.name }}</small></a>
      <button
        class="btn btn-danger btn-sm"
        (click)="auth.logout()">Log Out</button>
    </span>
  </div>
</nav>
```

The Header component handles the following:

* Navigation from the homepage to the dinosaurs list page
* Login button shown to unauthenticated users
* Profile picture, name, link to Profile page, and logout button shown to authenticated users

## Add Header Component to App Component Template

We want the Header component to show on every route. Open the `app.component.html` template file:

```html
<!-- src/app/app.component.html -->
<app-auth-header></app-auth-header>
<div class="container">
  <router-outlet></router-outlet>
</div>
```

The Header should now be visible in our app, allowing users to log in. Try it out!

