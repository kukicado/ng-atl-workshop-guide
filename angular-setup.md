# Angular Setup

> **Source Code**: The sample Angular application can be found at [**this GitHub repo**](https://github.com/kmaida/angular-auth).

We installed the Angular CLI in the [Dependencies](//dependencies.md) section earlier. Now we will use the CLI to generate our Angular app.

## Create a New Angular App

In a directory of your choice, run the following commands:

```bash
ng new auth0-app --routing --skip-tests --inline-style --inline-template
# (choose CSS when prompted)
```

The `ng new` command creates a new Angular application with routing and no app component tests.

> **Note**: We will not cover testing in this workshop. If you'd like to write your own tests, you should _not_ use the_ _`--skip-tests` flag.

## Add Bootstrap CSS

For ease of styling, add [Bootstrap CSS](https://getbootstrap.com/docs/4.0/getting-started/introduction/#css) to your `index.html` file's `<head>` like so:

```html
<!-- src/index.html -->
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.2.1/css/bootstrap.min.css" 
  rel="stylesheet" integrity="sha384-GJzZqFGwb1QTTN6wy59ffF1BuGJpLSa9DkKMp0DgiMDm4iYMj70gZWKYbI706tWS" 
  crossorigin="anonymous">
```

## Add Dependencies

Add dependencies for Node server:

```bash
npm install express path --save
```

Create `/server.js` file and add this code:

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

Add `stage` and `prod` NPM scripts to the `/package.json` file:

```json
// package.json
{
  ...
  "scripts": {
    ...,
    "stage": "ng build --configuration=stage && NODE_ENV=stage node server",
    "prod": "ng build --prod && node server"
  },
  ...
}
```

Open `/angular.json` and add a `stage` environment to the Angular app like so:

```json
// angular.json
{
  ...,
          "configurations": {...,
            "stage": {
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.stage.ts"
                }
              ],
              "optimization": true,
              "outputHashing": "all",
              "sourceMap": false,
              "extractCss": true,
              "namedChunks": false,
              "aot": true,
              "extractLicenses": true,
              "vendorChunk": false,
              "buildOptimizer": true
            },
            ...
}
```

Add `/src/environments/environment.stage.ts` file. Its contents should initially look like this:

```js
// environment.stage.ts
export const environment = {
  production: false,
  envName: 'stage'
};
```

(When we're all finished with the workshop, contents should look like [environment.stage.ts.sample](https://github.com/kmaida/angular-auth/blob/master/src/environments/environment.stage.ts.sample).)

We are now ready to start building our Angular application.

