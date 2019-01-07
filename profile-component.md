# Profile Component

The Profile component in this sample app is going to be very basic. All it will do is show the user's profile data in a relatively unformatted manner. The route for the Profile page component is protected by the [Authentication guard](/route-guards.md#authentication-guard) so that only authenticated users should be able to access it.

> **Note**: Feel free to revisit this component later and make it look much nicer.

## Profile Component Class

Open the `profile.component.ts` file:

```js
// src/app/pages/profile/profile.component.ts
import { Component, OnInit } from '@angular/core';
import { Title } from '@angular/platform-browser';
import { AuthService } from './../../auth/auth.service';
import { throwError } from 'rxjs';
import { tap, catchError } from 'rxjs/operators';

@Component({
  selector: 'app-profile',
  templateUrl: './profile.component.html',
  styles: []
})
export class ProfileComponent implements OnInit {
  user$ = this.auth.userProfile$.pipe(
    catchError(err => throwError(err)),
    tap(
      user => this.loading = false,
      err => {
        this.loading = false;
        this.error = true;
      }
    )
  );
  loading = true;
  error: boolean;

  constructor(
    private title: Title,
    private auth: AuthService
  ) { }

  ngOnInit() {
    this.title.setTitle('Profile');
  }

}
```

The [title](https://angular.io/api/platform-browser/Title) of our profile page should be the user's name. We'll create a `profileKeys` array consisting of the keys from our user's profile object. This way, we can iterate over the array to list out all of the user's profile data without manually adding each property in the template.

## Profile Component Template

Open the `profile.component.html` file:

```html
<!-- src/app/pages/profile/profile.component.html -->
<ng-template #noProfile>
  <app-loading *ngIf="loading"></app-loading>
  <app-error *ngIf="error"></app-error>
</ng-template>

<ng-container *ngIf="(user$ | async) as user else noProfile">
  <h2>{{ user.name }}</h2>
  <ul>
    <li *ngFor="let prop of user | keyvalue">
      <strong class="pr-1">{{ prop.key }}:</strong><code>{{ prop.value | json}}</code>
    </li>
  </ul>
</ng-container>
```

We can use the [`keyvalue` pipe](https://angular.io/api/common/KeyValuePipe) to iterate over the `user` array to output each key/value pair from the `auth.userProfile` and display the values cleanly using the [JSON pipe](https://angular.io/api/common/JsonPipe).

