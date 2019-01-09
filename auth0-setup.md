# Auth0 Setup

We need to configure our new Auth0 account to set up an [API](#create-an-api), [client](#create-a-client), [social identity providers](#set-up-social-identity-providers), and a [rule](#user-roles-rule). Once we've set up Auth0 and added it to our Angular app, users will be presented with a [login page](https://auth0.com/docs/hosted-pages/login) to sign in:  
![](https://cdn.auth0.com/blog/resources/auth0-centralized-login.jpg)

Let's set this up! Sign into your [**Auth0 Dashboard**](https://manage.auth0.com) to proceed.

## Create an API {#create-an-api}

Create a new Auth0 API by navigating to the [APIs](https://manage.auth0.com/#/apis) section in the Auth0 Dashboard sidebar and clicking the **+Create API** button. Your new API should have the following settings:

* **Name**: a suitable name of your choice
* **Identifier**: this should be the [URL for our API](/node-api.md#serve-the-api) \(_cannot_ be changed later\), `http://localhost/3001/api/`
* **Signing Algorithm**: `RS256`

![](/assets/api-setup.png)

Click the **Create** button to save your new API. Make note of the **Identifier**. We will need this setting in both our [Node.js API](/node-api.md#configuration) and our [Angular app configuration](/angular-setup.md#configure-environment) \(it will be our `audience` value\).

In your newly created API navigate to the **Scopes** tab and add the following scopes:

* `read:dino-details`
* `write:dino-fav`

![](/assets/add_scopes.png)

We will use these scopes to delegate API access to our Angular application.

## Create an Application {#create-a-client}

Auth0 _applications_ represent clients; in this case, our Angular front-end. Create a new application by navigating to the [Applications](https://manage.auth0.com/#/applications) item in the sidebar and clicking the **+Create Application** button. You should see the following screen:

![](https://cdn.auth0.com/blog/ngatl/new-client.jpg)

Give your new Application a suitable name \(such as `Auth0 App`\) and select **Single Page Web Applications** as the client type. Then click the **Create** button.

Once your Application has been created, select the **Settings** tab and make note of the **Domain** and **Client ID**. You'll need these settings to configure Auth0 in your Angular application.

Add the following configuration to the settings for this client:

* **Allowed Callback URLs**: `http://localhost:4200/callback`
* **Allowed Web Origins**: `http://localhost:4200`
* **Use Auth0 instead of the IdP to do Single Sign On**: `On`

> **Note**: If you're using an older Auth0 client app that you created at an earlier date, scroll down to the bottom of the **Settings** tab and click **Show Advanced Settings**. Select the **OAuth** tab and verify that the **JsonWebToken Signature Algorithm** is set to `RS256`. \(New clients use this algorithm by default, but older ones may not be set.\)

## Set Up Social Identity Providers {#set-up-social-identity-providers}

You can set up [social identity providers](https://auth0.com/docs/identityproviders#social) \(IdPs\) if you wish to allow your users to log in with providers like Google, Facebook, Twitter, and more. In the Auth0 Dashboard sidebar, navigate to **Connections** &gt; [**Social Connections**](https://manage.auth0.com/#/connections/social).

Switch on any social identity provider you wish you use in your app. You should _not_ use the default Auth0 dev key. Instead, you should set up your own social provider app. You can do this by clicking a provider's **How to obtain a ClientID?** link.

![](https://cdn.auth0.com/blog/ngatl/google-idp.png)

**Then follow the instructions to obtain an OAuth 2.0 Client ID for that social provider and fill it in.**

> **Important Note**: Social providers using Auth0 dev keys are identified by an orange `!` icon and a warning dialog in the main view. You should obtain non-development client IDs because Auth0 dev keys are _not_ acceptable in a production environment. In addition, using dev keys will _prevent_ silent token renewal from succeeding. Set up your own apps with the social identity providers to avoid this.

Once you've enabled a social provider and provided your own Client ID, select that provider's **Applications** tab in Auth0 and enable the IdP for the [Auth0 application you created for this project](#create-a-client). **Save** your settings.

