# Dinosaurs Component

Now it's time to call our API and display data to authenticated users in our Dinosaurs component.

## Dinosaurs Component Class

Next open the `dinosaurs.component.ts` class file:

```js
// src/app/pages/dinosaurs/dinosaurs.component.ts
import { Component, OnInit } from '@angular/core';
import { Title } from '@angular/platform-browser';
import { AuthService } from './../../auth/auth.service';
import { ApiService } from './../../data/api.service';
import { tap } from 'rxjs/operators';

@Component({
  selector: 'app-dinosaurs',
  templateUrl: './dinosaurs.component.html',
  styles: []
})
export class DinosaursComponent implements OnInit {
  loading = true;
  error: boolean;
  dinos$ = this.api.getDinos$().pipe(
    tap(
      res => this.loading = false,
      err => {
        this.loading = false;
        this.error = true;
      }
    )
  );
  pageTitle = 'Dinosaurs';

  constructor(
    private title: Title,
    public auth: AuthService,
    private api: ApiService
  ) { }

  ngOnInit() {
    this.title.setTitle(this.pageTitle);
  }

}
```

## Dinosaurs Component Template

Next open the `dinosaurs.component.html` template file:

```html
<!-- src/app/pages/dinosaurs/dinosaurs.component.html -->
<ng-template #noDinos>
  <app-loading *ngIf="loading"></app-loading>
  <app-error *ngIf="error"></app-error>
</ng-template>

<div *ngIf="(dinos$ | async) as dinos else noDinos">
  <h1 class="text-center py-2">🦕 {{ pageTitle }} 🦖</h1>
  <p *ngIf="!auth.isAuthenticated" class="text-center">
    Log in to learn more about dinosaurs!
  </p>
  <ng-container *ngFor="let dino of dinos">
    <app-dino [dino]="dino"></app-dino>
  </ng-container>
</div>
```

We're using the [Async pipe](https://angular.io/api/common/AsyncPipe) declaratively in our template. This unwraps a value from our `dinosaurs$` observable and will set up subscription and unsubscribing automatically. We'll also use the [NgIf directive](https://angular.io/api/common/NgIf#showing-an-alternative-template-using-else) and a [template reference variable](https://angular.io/guide/template-syntax#template-reference-variables--var-) to show the Loading component or Error component if `dinosaurs$ | async` has not emitted a value. The resulting value is set as `dino`, and we can then use the [NgFor directive](https://angular.io/api/common/NgForOf) to iterate over the array of dinosaurs and display each one.

> **Note**: You can test error handling in your app by stopping the Node API server. This will cause API requests to fail in the Angular app, and it should display errors appropriately.

## Dino Component Class

We are also using the `app-dino`child  component to display additional details about the dinosaurs. Let's go ahead and implement that code as well.

```js
import { Component, OnInit, Input } from '@angular/core';
import { IDino } from '../../../data/dino.interface';
import { AuthService } from './../../../auth/auth.service';

@Component({
  selector: 'app-dino',
  templateUrl: './dino.component.html',
  styles: []
})
export class DinoComponent implements OnInit {
  @Input() dino: IDino;

  constructor(public auth: AuthService) { }

  ngOnInit() {
  }

}
```

## Dino Component Template

```html
<div class="card my-2">
  <div class="card-body">
    <h5 class="card-title">
      {{ dino.name }} <span *ngIf="dino.favorite">⭐️</span>
    </h5>
    <div class="card-text">
      <p class="mb-0">
        <em>({{ dino.pronunciation }})</em>
      </p>
      <a
        *ngIf="auth.isAuthenticated"
        class="btn btn-primary btn-sm mt-2"
        routerLink="/dinosaur/{{ dino.name | lowercase }}">Learn More</a>
    </div>
  </div>
</div>
```







