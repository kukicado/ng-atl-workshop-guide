# Token Interceptor Service

While we could pass write code to add the Authorization header to our HTTP client requests in our API service, we would quickly start violating the DRY principle of programming. An Angular [HTTP interceptor](https://angular.io/api/common/http/HttpInterceptor) 
The final piece of our Authentication module is a service that provides an HTTP interceptor will attach the authenticated user's JWT access token to every API request. Let's see what that code looks like:

```js
// src/app/auth/secure-interceptor.service.ts
import { Injectable } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor
} from '@angular/common/http';
import { AuthService } from './auth.service';
import { Observable, throwError } from 'rxjs';
import { filter, mergeMap, catchError } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class SecureInterceptor implements HttpInterceptor {

  constructor(private auth: AuthService) { }

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    if (req.url.indexOf('/secure/') > -1) {
      return this.auth.accessToken$.pipe(
        filter(token => typeof token === 'string'),
        mergeMap(token => {
          const tokenReq = req.clone({
            setHeaders: { Authorization: `Bearer ${token}` }
          });
          return next.handle(tokenReq);
        }),
        catchError(err => throwError(err))
      );
    }
    return next.handle(req);
  }
}
```

This service _intercepts_ each outgoing HTTP request. It clones the request and adds an `Authorization: Bearer {jwt}` header using the authenticated user's access token.

## Use the Token Interceptor

Just like our Auth Guard, we'll have to tell our Angular application to use our created interceptor if we want it to work. Let's go back to our routes and change them to look like the following:

```js
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { SecureInterceptor } from './auth/secure-interceptor.service';
import { AuthGuard } from './auth/auth.guard';
import { HomeComponent } from './pages/home/home.component';
import { CallbackComponent } from './pages/callback.component';
import { DinosaursComponent } from './pages/dinosaurs/dinosaurs.component';
import { DinosaurDetailsComponent } from './pages/dinosaur-details/dinosaur-details.component';
import { ProfileComponent } from './pages/profile/profile.component';

const routes: Routes = [
 // .. 
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: SecureInterceptor,
      multi: true
    }
  ]
})
export class AppRoutingModule { }
```

Now we have added our interceptor service ensuring that whenever we try to access a **secure** route, our Angular application automatically attached the `access_token` to the Authorization header.

