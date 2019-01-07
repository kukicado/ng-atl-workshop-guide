# Loading and Error Components

The Loading and Error components are shared components.

## Loading Component

The Loading component simply shows a loading image. Let's add this image to our project.

Download the following image and save it here: `src/assets/images`

![](/assets/loading.svg)

Next open the `loading.component.ts` file. This is a flat component file with inline template and styles. Add the following code:

```js
// src/app/shared/loading.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-loading',
  template: `
    <div class="text-center my-3">
      <img src="/assets/images/loading.svg" alt="Loading...">
    </div>
  `,
  styles: []
})
export class LoadingComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

}
```

## Error Component

The Error component is another shared component. Open the `error.component.ts` file and add:

```js
// src/app/shared/error.component.ts
import { Component, OnInit, Input } from '@angular/core';

@Component({
  selector: 'app-error',
  template: `
    <p class="alert alert-danger">
      <strong class="pr-1">Oops!</strong>
      <ng-container *ngIf="errorMsg else defaultErr">{{ errorMsg }}</ng-container>
      <ng-template #defaultErr>Something went wrong! Please try again.</ng-template>
    </p>
  `,
  styles: []
})
export class ErrorComponent implements OnInit {
  @Input() errorMsg: string;

  constructor() { }

  ngOnInit() {
  }

}
```

The Error component uses [input binding](https://angular.io/guide/component-interaction#pass-data-from-parent-to-child-with-input-binding) to accept a `[errorMsg]` that can be passed in to provide the error text. If no `errorMsg` is passed to the component, it will show a default message.

