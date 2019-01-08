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
