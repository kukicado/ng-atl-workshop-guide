# Dino-Details Component

Let's now implement the final part of our application, the Dinosaur Details Component. Dino Details will display additional information about the selected dinosaur for authenticated and priviledged users. Editors will also be able to favorite dinosaurs.

## Dino-Details Component Class

Next open the `dinosaur-details.component.ts` class file and add the following code:

```js
// src/app/pages/dinosaur-details/dinosaur-details.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Title } from '@angular/platform-browser';
import { ActivatedRoute } from '@angular/router';
import { tap } from 'rxjs/operators';
import { Subscription } from 'rxjs';
import { IDinoDetails } from '../../data/dino.interface';
import { ApiService } from './../../data/api.service';
import { AuthService } from 'src/app/auth/auth.service';

@Component({
  selector: 'app-dinosaur-details',
  templateUrl: './dinosaur-details.component.html',
  styles: []
})
export class DinosaurDetailsComponent implements OnInit, OnDestroy {
  name: string;
  params$ = this.route.params.pipe(
    tap(
      params => this.dinoSetup(params['name']),
      err => this.handleErr('No dinosaur found.')
    )
  );
  loading = true;
  error: boolean;
  errorMsg: string;
  dinoSub: Subscription;
  dino: IDinoDetails;
  toggleFavSub: Subscription;
  savingFav: boolean;

  constructor(
    private title: Title,
    private route: ActivatedRoute,
    private api: ApiService,
    public auth: AuthService
  ) { }

  ngOnInit() {
  }

  private dinoSetup(nameParam: string) {
    this.dinoSub = this.api.getDinoByName$(nameParam).subscribe(
      dino => {
        if (dino) {
          this.title.setTitle(dino.name);
          this.loading = false;
          this.dino = dino;
        } else {
          this.handleErr('The dinosaur you requested could not be found.');
        }
      },
      err => this.handleErr()
    );
  }

  private handleErr(msg?: string) {
    this.error = true;
    this.loading = false;
    if (msg) {
      this.errorMsg = msg;
    }
  }

  toggleFav() {
    this.savingFav = true;
    this.toggleFavSub = this.api.favDino$(this.dino.name).subscribe(
      dino => {
        this.dino = dino;
        this.savingFav = false;
      }
    );
  }

  get getFavBtnText() {
    if (this.savingFav) {
      return 'Saving...';
    }
    return !this.dino.favorite ? 'Favorite' : 'Un-favorite';
  }

  ngOnDestroy() {
    if (this.toggleFavSub) {
      this.toggleFavSub.unsubscribe();
    }
    this.dinoSub.unsubscribe();
  }

}
```

## Dino-Details Component Template

Next open the `dinosaur-details.component.html` template file:

```html
<!-- src/app/pages/dinosaur-details/dinosaur-details.component.html -->
<ng-template #noDino>
  <app-loading *ngIf="loading"></app-loading>
  <app-error *ngIf="error" [errorMsg]="errorMsg"></app-error>
</ng-template>

<ng-container *ngIf="(params$ | async) && dino else noDino">
  <h1 class="text-center py-2">{{ dino.name }} <span *ngIf="dino.favorite">⭐️</span></h1>
  <div class="card my-2">
    <div class="card-body">
      <div class="card-text">
        <ul class="list-unstyled">
          <li><strong>Pronunciation:</strong> {{ dino.pronunciation }}</li>
          <li><strong>Meaning of Name:</strong> "{{ dino.meaningOfName }}"</li>
          <li><strong>Lived:</strong> {{ dino.period }} ({{ dino.mya }} million years ago)</li>
          <li><strong>Diet:</strong> {{ dino.diet }}</li>
          <li><strong>Length:</strong> {{ dino.length }}</li>
        </ul>
      </div>
      <p class="card-text lead" [innerHTML]="dino.info"></p>
      <p>
        <button
          *ngIf="auth.userHasRole('editor')"
          class="btn btn-sm"
          [ngClass]="{ 'btn-success': !dino.favorite, 'btn-danger': dino.favorite }"
          [disabled]="savingFav"
          (click)="toggleFav()">{{ getFavBtnText }}</button>
      </p>
    </div>
  </div>
</ng-container>
```
