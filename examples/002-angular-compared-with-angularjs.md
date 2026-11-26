# Angular Compared with AngularJS: Examples

This example builds the same tiny counter in AngularJS 1.x and modern Angular. It highlights the different application structure and template APIs.

Read [Chapter 2](../lessons/002-angular-compared-with-angularjs.md) first, and use the [Q&A](../qnas/002-angular-compared-with-angularjs.md) to review the terminology.

## What you will build

Both versions show a count that starts at zero. Selecting **Add one** increases it by one.

Expected behavior:

- Initial display: Count: 0
- After one click: Count: 1
- After three clicks: Count: 3

The output is intentionally small so the framework differences are easy to see.

## Example A: AngularJS 1.x

This is a legacy-style AngularJS example. It assumes an AngularJS 1.x library is already available to the page as vendor/angular/angular.min.js. Use the copy and version managed by the existing project; AngularJS is end-of-life and this example is for learning or maintaining legacy code, not for starting a new application.

### Files

~~~text
angularjs-counter/
├── index.html
├── app.js
└── vendor/
    └── angular/
        └── angular.min.js
~~~

### index.html

~~~html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>AngularJS Counter</title>
</head>
<body ng-app="counterApp">
  <main ng-controller="CounterController as vm">
    <h1>AngularJS counter</h1>
    <p>Count: {{ vm.count }}</p>
    <button type="button" ng-click="vm.increment()">Add one</button>
  </main>

  <script src="vendor/angular/angular.min.js"></script>
  <script src="app.js"></script>
</body>
</html>
~~~

### app.js

~~~js
angular
  .module('counterApp', [])
  .controller('CounterController', function CounterController() {
    const vm = this;

    vm.count = 0;
    vm.increment = function increment() {
      vm.count += 1;
    };
  });
~~~

### How the AngularJS version works

1. The ng-app attribute tells AngularJS which application module to bootstrap.
2. The ng-controller attribute connects the controller to the main element.
3. The alias vm exposes controller properties and methods to the template.
4. The interpolation expression displays the count.
5. The ng-click directive calls the controller method when the button is selected.
6. AngularJS's binding and digest mechanisms update the displayed value.

The controller-as style avoids adding this example's values directly to $scope. A legacy project may use $scope instead; follow its existing style when maintaining it.

## Example B: Modern Angular

Create a minimal modern Angular application with the Angular CLI. The following command creates a standalone application with routing disabled for this small example:

~~~sh
ng new angular-counter --standalone --routing=false --style=css
cd angular-counter
~~~

Replace or create the root component and application entry point with the files below. A standalone root component can be bootstrapped directly.

### src/app/counter.component.ts

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: '<main><h1>Angular counter</h1><p>Count: {{ count() }}</p><button type="button" (click)="increment()">Add one</button></main>',
})
export class CounterComponent {
  readonly count = signal(0);

  increment(): void {
    this.count.update((currentCount) => currentCount + 1);
  }
}
~~~

### src/main.ts

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { CounterComponent } from './app/counter.component';

bootstrapApplication(CounterComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

Start the development server from the project directory:

~~~sh
ng serve
~~~

Open the local address printed by the CLI. The count should behave like the AngularJS version.

### How the Angular version works

1. The Angular CLI compiles and serves the application.
2. The main entry point bootstraps the root component.
3. Component metadata declares the selector and template.
4. The signal stores the current count.
5. The event binding calls the class method when the button is selected.
6. The template reads the signal by calling it, and Angular updates the view when it changes.

The explicit standalone setting is supported in modern Angular versions that provide standalone APIs. In Angular 19 and later, standalone is the default for components, but keeping the setting in this teaching example makes the intent clear.

## Compare the two implementations

| Concern | AngularJS example | Modern Angular example |
| --- | --- | --- |
| Startup | ng-app bootstraps an AngularJS module | main.ts calls bootstrapApplication |
| UI boundary | Controller attached to a template | Component class with metadata and a template |
| State | Controller property vm.count | Signal read as count() |
| Click behavior | ng-click invokes vm.increment() | (click) invokes increment() |
| Display | Interpolation reads vm.count | Interpolation reads count() |
| Framework updates | AngularJS binding and digest processing | Angular change detection with signal tracking |
| Tooling | HTML page loads AngularJS and app scripts | Angular CLI builds and serves the application |

The repeated idea is a counter with state and a button. The architecture and APIs differ. This is why migration involves more than changing directive names.

## Migration exercise

Suppose the AngularJS counter is part of a larger screen.

1. Identify which template area owns the counter and which code changes its value.
2. Record any behavior beyond incrementing, such as reset, persistence, or validation.
3. Decide whether the counter is local UI state or shared application state.
4. Implement that behavior in a modern Angular component or an appropriate shared service.
5. Compare the old and new behavior using the same user actions.
6. Remove or retire the AngularJS implementation only after the replacement and its integrations are verified.

For this simple example, a signal is a good local state choice. A larger application may need a service, RxJS, or another state design. Choose based on ownership and sharing requirements rather than converting every old property mechanically.

## Optional variations

### Add a reset button in modern Angular

Add this method to the component class:

~~~ts
reset(): void {
  this.count.set(0);
}
~~~

Add this button to the template:

~~~html
<button type="button" (click)="reset()">Reset</button>
~~~

The signal's set method assigns a value directly. The update method used earlier derives a new value from the current one.

### Add the same reset behavior in AngularJS

Add this method inside the controller:

~~~js
vm.reset = function reset() {
  vm.count = 0;
};
~~~

Then add the button to the template:

~~~html
<button type="button" ng-click="vm.reset()">Reset</button>
~~~

Both versions can express the same behavior, but their controller/component structure and update APIs remain different.

## Troubleshooting

### AngularJS displays an unknown directive or a blank page

Confirm the AngularJS library script loads before app.js, the paths are correct, and the browser console has no script errors. Also confirm the module name in ng-app matches the module created in app.js.

### The AngularJS button does not change the displayed value

Confirm the controller alias is vm in both the template and the controller-as declaration, and that the button calls vm.increment(). Check the browser console for an exception.

### Angular reports that it cannot find the component import

Check the path and filename case. The import in main.ts must match the component file path.

### The Angular template shows an error near the count expression

For a signal, the template must read the value by calling it, as in count(). The component property itself is not the displayed number.

### The CLI generated a different root component layout

Angular CLI templates can vary by version and options. Keep the project configuration generated by the CLI, then make sure main.ts bootstraps the component that is exported from the file you edited.

## Key takeaway

AngularJS and modern Angular can implement the same user-facing feature, but they use different application structures and APIs. Compare behavior and responsibilities, then rebuild each feature using the modern framework's component model and tools.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="001-angular-and-component-based-web-applications.md">001 - Angular and Component-Based Web Applications</a></td><td><a href="../lessons/002-angular-compared-with-angularjs.md">Lesson</a></td><td><a href="../qnas/002-angular-compared-with-angularjs.md">Q&amp;A</a></td><td><a href="003-browser-dom-http-and-javascript-foundations.md">003 - Browser, DOM, HTTP, and JavaScript Foundations</a></td>
</tr>
  </tbody>
</table>
