# API Service

In this section we will glue our Angular and Node applications together. We'll do this in two parts. The first part will have us create a dinosaur interface so that we have a well defined contract or data structure when in comes to dealing with our dinosaurs. In the second part, we'll actually write the calls that will be made to our Node API.

## Configuration

Before we get to implementing our API Service, let's set a base URL for our API. This will ensure we don't repeat ourselves too much. Open up the `environment.ts` file and add the following code:

```js
export const environment = {
  production: false,
  api: {
    baseUrl: '{YOUR_API_URL}' // e.g., http://localhost:3005/api/
  }
};
```

Next, since we need to talk to an external API over HTTP we are going to need to include the HTTPClientModule to our Angular app. To do this, add the following to your `app.module.ts` file:

```ts
import {HttpClientModule} from '@angular/common/http';

...

imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule
  ],
```

## Dino Interface

An interface in the simplest terms allow us to define what our data structures will look like. Interfaces are a TypeScript feature but since Angular is written in TypeScript we'll make use of them here. We'll define two interfaces that will represent two different dinosaur data structures. `IDino` will be a simple data structure that simply has the dinosaurs name, pronounciation, and whether it has been favorited or not, while the `IDinoDetails` interface will have all of the details of our dinosaur. We'll implement them like so:

```js
interface IDino {
  name: string;
  pronunciation: string;
  favorite?: boolean;
}
interface IDinoDetails {
  name: string;
  pronunciation: string;
  meaningOfName: string;
  diet: string;
  length: string;
  period: string;
  mya: string;
  info: string;
  favorite?: boolean;
}

export { IDino, IDinoDetails };
```


## HTTP Client

Let's implement the service logic that will interact with our Node API. Open the `api.service.ts` file and add the following code:

```js
// src/app/shared/api.service.ts
import { Injectable } from '@angular/core';
import { throwError, Observable } from 'rxjs';
import { HttpClient } from '@angular/common/http';
import { catchError, tap } from 'rxjs/operators';
import { IDino, IDinoDetails } from './dino.interface';
import { environment } from './../../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class ApiService {

  constructor(private http: HttpClient) { }

  getDinos$(): Observable<IDino[]> {
    return this.http
      .get<IDino[]>(`${environment.api.baseUrl}dinosaurs`)
      .pipe(
        catchError(err => throwError(err))
      );
  }

  getDinoByName$(name: string): Observable<IDinoDetails> {
    return this.http
      .get<IDinoDetails>(`${environment.api.baseUrl}secure/dinosaur/${name}`)
      .pipe(
        catchError(err => throwError(err))
      );
  }

  favDino$(name: string): Observable<IDinoDetails> {
    return this.http
      .post<IDinoDetails>(`${environment.api.baseUrl}secure/fav`, { name: name })
      .pipe(
        tap(res => console.log(name + ' marked as favorite!')),
        catchError(err => throwError(err))
      );
  }
}
```

We are using Angular HTTP Client library to create three methods that correspond with our three Node API endpoints. Although all of our HTTP requests require authorization, we don't need to add the `Authorization` header in the API service. Instead, the [token interceptor service](/token-interceptor-service.md) will take care of that.

