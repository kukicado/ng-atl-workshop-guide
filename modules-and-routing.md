# Routing

Now that we've generated our app's architecture, let's set up our routing properly. Open up the `app-routing.module.ts` file and add the following code:

```js
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { AuthService } from './auth/auth.service';
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { SecureInterceptor } from './auth/secure-interceptor.service';
import { AuthGuard } from './auth/auth.guard';
import { HomeComponent } from './pages/home/home.component';
import { CallbackComponent } from './pages/callback.component';
import { DinosaursComponent } from './pages/dinosaurs/dinosaurs.component';
import { DinosaurDetailsComponent } from './pages/dinosaur-details/dinosaur-details.component';
import { ProfileComponent } from './pages/profile/profile.component';

const routes: Routes = [
  {
    path: 'callback',
    component: CallbackComponent
  },
  {
    path: 'dinosaurs',
    component: DinosaursComponent
  },
  {
    path: 'dinosaur/:name',
    component: DinosaurDetailsComponent,
    canActivate: [
      AuthGuard
    ]
  },
  {
    path: 'profile',
    component: ProfileComponent,
    canActivate: [
      AuthGuard
    ]
  },
  {
    path: '',
    component: HomeComponent,
    pathMatch: 'full'
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
  providers: [
    AuthService,
    {
      provide: HTTP_INTERCEPTORS,
      useClass: SecureInterceptor,
      multi: true
    }
  ]
})
export class AppRoutingModule { }
```

The home, dinosaurs and callback pages are publicly accessible. The dinosaur detail and profile components on the other hand [can activate](https://angular.io/api/router/CanActivate) if the user is authenticated.

> **Note**: Don't be alarmed if your app doesn't compile right now. We still have to make some changes in the files that we've generated in order for everything to fit together.



