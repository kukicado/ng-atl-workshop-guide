# Production Configuration

Our app development is complete! Now we'll discuss how to get the app production-ready.

## Add Web Server to Angular Project

Let's create a simple web server in our Angular project folder to serve our Angular app from the /dist folder. This is where our built files will be outputted.

## Web Server Dependencies

First we'll install the web server's dependencies with a package manager:

```bash
npm install express --save
```

## Server File

Next create a server.js file in the root of the Angular project folder (this should be next to the src folder, not inside it). Open the file and add:

```js
// Dependencies
const express = require('express');
const path = require('path');
// App
const app = express();

// Security middleware
function resSec(req, res, next) {
  if (app.get('env') !== 'stage') {
    // HTTP Strict Transport Security (HSTS)
    // Enforces HTTPS across the entire app
    // While nginx can do a redirect, HSTS redirects
    // before anything is sent to the server
    // (Only run this in an SSL environment)
    res.setHeader('Strict-Transport-Security', 'max-age=630720');
  }
  // Defend against Cross Site Scripting (XSS)
  // This is when a malicious entity injects scripts
  res.setHeader('X-XSS-Protection', '1; mode=block');
  // Require iFrame sources to come from the same origin
  res.setHeader('X-Frame-Options', 'SAMEORIGIN');
  // Content Security Policy
  // Preventing XSS to ensure scripts only come
  // from approved origins
  res.setHeader("Content-Security-Policy", "script-src 'self'");
  // Send the request on with security headers
  return next();
}
app.use(resSec);

// Set static path to Angular app
app.use('/', express.static(path.join(__dirname, './dist/angular-auth')));
// Pass routing to Angular app
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, './dist/angular-auth/index.html'));
});

// Server
const port = '3000';
app.listen(port, () => console.log(`Server running on localhost:${port}`));
```

## Production Environment Variables

When we build our project for deployment, the Angular CLI will look for and use the environment.prod.ts file, which we haven't configured yet. Let's do so now:

```js
// src/environments/environment.prod.ts
export const environment = {
  production: true,
  auth: {
    clientId: '{Auth0_Client_ID}',
    domain: '{Auth0_Domain}' // e.g., 'your-account.auth0.com',
    redirect: 'http://localhost:8080/callback',
    audience: '{Auth0_API_Identifier}', // e.g., 'http://localhost:3001/api/'
    scope: 'openid profile email',
    namespace: '{Auth0_Rules_Roles_Namespace}' // e.g., 'https://example.com/roles'
  }
};
```

You can copy your configuration from environment.ts for now, and just update the redirect to point to your Angular web server on port 8080 (instead of the development server on port 4200).

> Note: For simplicity, we will use the same client and API that we used in development.

## Build App

Now we can build our Angular app for production. Do so using this command:

```
ng build --prod
```

The `--prod`` meta-flag does the following:

* Uses Ahead-of-Time (AoT) compilation (compiles the app at build time instead of runtime)
* Bundles, minifies, and uglifies code
* Performs dead code elimination
* Uses build optimizer for improved tree-shaking and smaller bundles (with Angular 5 and AoT)

The production build files will be saved in a dist folder in the root and the build command will use the configuration in the `environment.prod.ts` file.

## Serve App

We can now run our production app build by starting the web server like so:

```
node server
```

Additionally, we can build and serve our app by adding a new NPM script to our `package.json` file. Add the following:

Add `stage` and `prod` NPM scripts to the `/package.json` file:

```json
// package.json
{
  ...
  "scripts": {
    ...,
    "prod": "ng build --prod && node server"
  },
  ...
}
```

Now you can simple run `npm run prod` and your application will build, and then automatically serve.

## Update Auth0 Client Configuration

Our app is now running at http://localhost:8080, but we won't be able to log in successfully yet. Our Auth0 client currently expects the application to reside at http://localhost:4200. We need to add some additional configuration in order to grant permission to Auth0 to redirect to the new port, and to accept it as an origin for token renewal.

In your Auth0 Dashboard, go to Clients and select the client you created for this project. In addition to the current development settings, add the following:

* **Allowed Callback URLs**: http://localhost:8080/callback
* **Allowed Web Origins:** http://localhost:8080

> Note: If or when you deploy your application to a non-localhost URL, you will need to add the appropriate URLs to these settings as well. Alternately, you could create a new client for production. In addition, you will need to create a new API with a production (non-localhost) identifier.

Click the **Save Changes** button.

## Login

You should now be able to log into the Angular production build running on the web server. You can see how we could now deploy our Angular application and Node.js API to production URLs. You may have your own production requirements.
