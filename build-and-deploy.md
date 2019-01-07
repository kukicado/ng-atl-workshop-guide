# Production Configuration

Our app development is complete! Now we'll discuss how to get the app production-ready.

## Add Web Server to Angular Project

Let's create a simple web server in our Angular project folder to serve our Angular app from the `/dist` folder. This is where our built files will be outputted.

### Web Server Dependencies

First we'll install the web server's dependencies with a package manager:

```
npm install express cors --save
```

### Server File

Next create a `server.js` file in the root of the Angular project folder \(this should be next to the `src` folder, _not_ inside it\). Open the file and add:

```js
// server.js
// Modules
const path = require('path');
const express = require('express');
const cors = require('cors');

// App
const app = express();
app.use(cors());

// Run the app with static files from the /dist directory
app.use(express.static(__dirname + '/dist'));

// For all GET requests, send back /dist/index.html
// This passes routing to the Angular app
app.get('*', function(req, res) {
  res.sendFile(path.join(__dirname + '/dist/index.html'));
});

// Set port
const port = process.env.PORT || '8080';
app.set('port', port);

// Start the app
app.listen(port, () => console.log(`Angular app deployed at localhost:${port}`));
```

> **Note**: Launching the web server right now will fail because there is no `dist` directory yet, and we also haven't configured our production environment settings. We'll do those things next.

## Production Environment Variables

When we build our project for deployment, the Angular CLI will look for and use the `environment.prod.ts` file, which we haven't configured yet. Let's do so now:

```js
// src/environments/environment.prod.ts
export const environment = {
  production: true,
  auth: {
    clientId: '{YOUR_AUTH0_CLIENT_ID}',
    domain: '{YOUR_AUTH0_DOMAIN}', // e.g., you.auth0.com,
    redirect: '{YOUR_PRODUCTION_URL}/callback',
    logoutUrl: '{YOUR_PRODUCTION_URL}',
    roles_namespace: '{YOUR_ROLES_NAMESPACE}' // e.g., https://secure-dino-api/roles
  },
  api: {
    baseUrl: '{YOUR_API_URL}' // e.g., http://api.myproductionurl.com/
  }
};
```

You can copy your configuration from `environment.ts` for now, and just update the `redirect` to point to your Angular web server on port `8080` \(instead of the development server on port `4200`\).

> **Note**: For simplicity, we will use the same client and API that we used in development.

## Build and Serve App

Now we can build our Angular app for production. Do so using this command:

```
ng build --prod
```

The [`--prod` meta-flag](https://github.com/angular/angular-cli/wiki/build#--dev-vs---prod-builds) does the following:

* Uses [Ahead-of-Time \(AoT\) compilation](https://angular.io/guide/aot-compiler) \(compiles the app at build time instead of runtime\)
* Bundles, minifies, and uglifies code
* Performs dead code elimination
* Uses build optimizer for improved tree-shaking and smaller bundles \(with Angular 5 and AoT\)

The production build files will be saved in a `dist` folder in the root and the build command will use the configuration in the `environment.prod.ts` file.

We can now run our production app build by starting the web server like so:

```
node server
```

## Update Auth0 Client Configuration

Our app is now running at [http://localhost:8080](http://localhost:8080), but we won't be able to log in successfully yet. Our Auth0 client currently expects the application to reside at `http://localhost:4200`. We need to add some additional configuration in order to grant permission to Auth0 to redirect to the new port, and to accept it as an origin for token renewal.

In your [Auth0 Dashboard](https://manage.auth0.com), go to [Clients](https://manage.auth0.com/#/clients) and select the [client you created for this project](/auth0-setup.md#create-a-client). In addition to the current development settings, _add_ the following:

* **Allowed Callback URLs**: `http://localhost:8080/callback`
* **Allowed Web Origins**:** **`http://localhost:8080`

> **Note**: If or when you deploy your application to a non-localhost URL, you will need to add the appropriate URLs to these settings as well. Alternately, you could create a new client for production. In addition, you will _need_ to create a new API with a production \(non-localhost\) identifier.

Click the **Save Changes** button.

## Log In

You should now be able to log into the Angular production build running on the web server. You can see how we could now deploy our Angular application and Node.js API to production URLs. You may have your own production requirements. **Tell us about your production requirements and let us know if you have any questions!**

