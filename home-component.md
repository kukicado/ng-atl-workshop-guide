# Home Component

Let's implement our Home page next.

## Home Component Class

Open the `home.component.ts` class file:

```js
// src/app/pages/home/home.component.ts
import { Component, OnInit } from '@angular/core';
import { Title } from '@angular/platform-browser';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styles: []
})
export class HomeComponent implements OnInit {
  pageTitle = 'Welcome';

  constructor(private title: Title) { }

  ngOnInit() {
    this.title.setTitle(this.pageTitle);
  }

}
```

The Home component class should set the page title and make the Authentication service publicly available for use in the template.

## Home Component Template

Open the `home.component.html` file:

```html
<!-- src/app/pages/home/home.component.html -->
<h1 class="text-center py-2">{{ pageTitle }}</h1>
<p>Welcome to Angular authentication!</p>
```

