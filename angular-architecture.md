# Angular Architecture

Next we'll use the CLI to generate our application's primary architecture. From the root of your Angular project, run each of the following commands to set up the app.

## Callback Page

Remember that we [configured our Auth0 application](/auth0-setup.md#create-a-client) to use a route located at `http://localhost:4200/callback` when redirecting back to our application after authentication with Auth0's login page. The Callback component will call the method that parses the authentication hash and sets up a user's session. We can create the component easily with the CLI like so:

```
ng g component pages/callback --flat
```

The `g` is a shortcut for `generate`.

> **Note**: We could also use `ng g c` to generate a component, but for the sake of clarity, this workshop will always use the full name of the type of file we wish to create.

Newly generated components are _declared_ in the nearest parent module. In this case, this is the `app.module.ts` because we haven't created any other modules yet.

## Home Page

Generate the Home page component, which will be available to all users regardless of authentication state, and is the default view for our app.

```
ng g component pages/home --inline-template false
```

## Dinosaurs Page

Generate the Dinosaurs page component, which will be accessible to everyone. This page will show a list of dinosaurs fetched from the API with our unprotected `/api/dinosaurs` endpoint.

```
ng g component pages/dinosaurs --inline-template false
```

## Dinosaur Details Page

Generate the Dinosaurs Detail page component, which will be accessible to authenticated users that contain the scope `read:dino-details`. This page will show a additional information on the selected dinosaur from our protected `/api/secure/dinosaur/:name` endpoint.

```
ng g component pages/dinosaur-details --inline-template false
```

## Dinosaur Info Component

Generate the component that will display information for each dinosaur from a list of dinosaurs. Additionally, an authenticated user will be able to favorite a dinosaur, granted they have the right scope of `write:dino-fav` and further the role of `editor`. This component will interact with the protected `/api/secure/fav` endpoint.

```
ng g component pages/dinosaurs/dino
```

## Profile Page

Generate the Profile page component, which will be accessible to authenticated users to show their [personal information, which is fetched from Auth0](https://auth0.com/docs/libraries/auth0js/v9#extract-the-authresult-and-get-user-info) using the `client.userInfo()` method from `auth0.js`.

```
ng g component pages/profile --inline-template false
```

## Shared Components

Now we'll create a new module. This Shared module will supply components that need to be shared across the app's other modules.

```
ng g component shared/loading 
ng g component shared/error
```

# Auth Services

Next we'll create an Authentication module. This module will supply our app with everything it needs to share regarding authentication, such as an auth service, token interceptor service, and route guards.

```
ng g service auth/auth 
ng g guard auth/auth 
ng g service auth/secure-interceptor
ng g component auth/auth-header --inline-template false
```

# Data Service

Finally, let's create a service and interface so that we can work with our API data.

```
ng g service data/api
ng g interface data/dino
```

## _Aside:_ Linting

As we develop the Angular app, we can lint our project using the following Angular CLI command, which utilizes [Codelyzer](https://github.com/mgechev/codelyzer):

```
ng lint
```

Run this command periodically as you're building out your app to catch and correct any errors or warnings and to follow best practices.

