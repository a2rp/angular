# Dynamic Components and Programmatic Rendering — Examples

This walkthrough lets a dashboard user load one known report widget on demand. It uses a ViewContainerRef insertion point, sets a typed component input, reacts to an output, and clears the component when it is dismissed.

## 1. Create the widget component

~~~ts
// report-widget.ts
import { Component, input, output } from '@angular/core';

@Component({
  selector: 'app-report-widget',
  template: '<section aria-labelledby="report-title"><h2 id="report-title">{{ title() }}</h2><p>Report data appears here.</p><button type="button" (click)="dismissed.emit()">Close report</button></section>'
})
export class ReportWidgetComponent {
  readonly title = input.required<string>();
  readonly dismissed = output<void>();
}
~~~

The widget has a small contract: its host supplies a title, and it emits a dismissal event. It does not locate or mutate its parent.

## 2. Mark the insertion point

~~~ts
// widget-host.directive.ts
import { Directive, ViewContainerRef, inject } from '@angular/core';

@Directive({
  selector: '[appWidgetHost]'
})
export class WidgetHostDirective {
  readonly viewContainer = inject(ViewContainerRef);
}
~~~

This directive exposes the container at the location where the component should appear.

## 3. Load and create the widget from its host

~~~ts
// dashboard-page.ts
import { Component, ComponentRef, ViewChild } from '@angular/core';
import { ReportWidgetComponent } from './report-widget';
import { WidgetHostDirective } from './widget-host.directive';

@Component({
  selector: 'app-dashboard-page',
  imports: [WidgetHostDirective],
  templateUrl: './dashboard-page.html'
})
export class DashboardPage {
  @ViewChild(WidgetHostDirective, { static: true })
  private host!: WidgetHostDirective;

  private currentReport: ComponentRef<ReportWidgetComponent> | undefined;

  showSalesReport(): void {
    const container = this.host.viewContainer;
    container.clear();

    const componentRef = container.createComponent(ReportWidgetComponent);
    componentRef.setInput('title', 'Sales report');
    componentRef.instance.dismissed.subscribe(() => {
      if (this.currentReport === componentRef) {
        container.clear();
        this.currentReport = undefined;
      }
    });

    this.currentReport = componentRef;
  }

  removeReport(): void {
    this.host.viewContainer.clear();
    this.currentReport = undefined;
  }
}
~~~

Calling clear destroys the old view so only one report widget remains. Setting the input immediately after creation gives the required input a value before the next render. The current component reference is kept only while its view is active.

~~~html
<!-- dashboard-page.html -->
<main>
  <h1>Dashboard</h1>
  <button type="button" (click)="showSalesReport()">
    Show sales report
  </button>
  <button type="button" (click)="removeReport()">
    Remove report
  </button>

  <ng-container appWidgetHost></ng-container>
</main>
~~~

The host owns the ComponentRef or its ViewContainerRef. When it no longer needs the widget, clear or destroy it. Do not retain references to components after destruction.

## 4. Optional: lazy-load the known widget module

If the widget code should not load until the user asks for it, use a static dynamic import:

~~~ts
async showSalesReport(): Promise<void> {
  const { ReportWidgetComponent } =
    await import('./report-widget');

  const container = this.host.viewContainer;
  container.clear();

  const componentRef = container.createComponent(ReportWidgetComponent);
  componentRef.setInput('title', 'Sales report');
  this.currentReport = componentRef;
}
~~~

Keep the import path in application code. Do not build a module path from user-provided strings. If the widget type is known ahead of time and only needs to be deferred, consider Angular's @defer block instead.

## 5. Alternative: render a known dynamic type in a template

Use NgComponentOutlet when the chosen component type is simple to express as template state:

~~~ts
import { Component, Type, signal } from '@angular/core';
import { NgComponentOutlet } from '@angular/common';
import { ReportWidgetComponent } from './report-widget';

@Component({
  selector: 'app-widget-preview',
  imports: [NgComponentOutlet],
  template: '<ng-container *ngComponentOutlet="widgetType()" />'
})
export class WidgetPreview {
  readonly widgetType = signal<Type<unknown> | null>(
    ReportWidgetComponent
  );
}
~~~

This is declarative and can be easier to follow than imperative component creation. Use the API supported by your Angular version for passing inputs and outputs to the outlet.

## 6. Simpler alternative: defer a known component

If the widget is known and only needs to load after an interaction, @defer may be enough:

~~~html
<button type="button" (click)="showDetails = true">
  Load details
</button>

@if (showDetails) {
  @defer {
    <app-report-widget title="Sales report" />
  } @placeholder {
    <p>Report has not been loaded.</p>
  } @loading {
    <p>Loading report…</p>
  } @error {
    <p>The report could not be loaded.</p>
  }
}
~~~

Use one approach that fits the requirement; avoid combining dynamic rendering and deferral without a clear reason.

## 7. Review behavior and cleanup

Try loading the widget twice, dismissing it from inside the widget, and removing it from the host. Verify that the old view is destroyed and that output subscriptions do not keep stale UI alive. For server-rendered routes, verify that dynamic rendering participates correctly in hydration.

## Further reading

- [Programmatic component rendering](https://angular.dev/guide/components/programmatic-rendering)
- [ViewContainerRef API](https://angular.dev/api/core/ViewContainerRef)
- [Deferrable views](https://angular.dev/guide/templates/defer)
