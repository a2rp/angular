# Dynamic Components and Programmatic Rendering

Most UI is easiest to understand when components appear directly in a template. Dynamic rendering is useful when the component type is selected at runtime, such as a dashboard widget chosen from a known registry, or when a host must insert and remove an Angular view programmatically.

Do not use dynamic rendering only to avoid writing a template. If the component is known at build time and the goal is to defer its code, Angular's @defer block is usually a simpler starting point.

## Learning outcomes

After this lesson, you should be able to:

- Choose among a normal template, @defer, NgComponentOutlet, and ViewContainerRef.
- Pass inputs and receive outputs from dynamically rendered components.
- Understand the view-container lifetime and destroy dynamically created views.
- Lazy-load a component type from an explicit allowlist.
- Avoid unsafe dynamic component selection from untrusted data.
- Recognize rendering and hydration constraints around programmatic DOM insertion.

## 1. Choose the least dynamic tool that fits

Use an ordinary component element when the component is known and always belongs in the template. Use template control flow when the choice is based on known states. Use @defer when the component is known but its code should load later.

Use NgComponentOutlet when the template should render a runtime-selected component type and the needed configuration is relatively declarative. Use ViewContainerRef when imperative logic needs to insert, replace, move, or remove Angular views.

The standalone createComponent function is for cases where you need more control over where a host element is attached, such as an overlay. It has more lifecycle and change-detection responsibilities than ViewContainerRef.

## 2. Use NgComponentOutlet for a declarative dynamic type

A dynamic component should come from a known, typed registry, not from an arbitrary value received from a server:

~~~ts
import { Component, Type, signal } from '@angular/core';
import { NgComponentOutlet } from '@angular/common';

@Component({
  selector: 'app-dashboard',
  imports: [NgComponentOutlet],
  template: '<ng-container *ngComponentOutlet="selectedWidget()" />'
})
export class DashboardComponent {
  readonly selectedWidget = signal<Type<unknown> | null>(null);
}
~~~

An application can assign a component class from its own allowlist. A real application should also provide the intended inputs and outputs using the APIs supported by its Angular version.

Do not accept a string from untrusted input and turn it into an import path or executable component. Map validated user choices to a finite set of known components.

## 3. Use ViewContainerRef for imperative insertion

ViewContainerRef is tied to a location in Angular's view tree. Creating a component through it inserts the host view at that location and Angular manages it as part of that container.

A marker directive provides an explicit insertion point:

~~~ts
import { Directive, ViewContainerRef, inject } from '@angular/core';

@Directive({
  selector: '[appWidgetHost]'
})
export class WidgetHostDirective {
  readonly viewContainer = inject(ViewContainerRef);
}
~~~

The host component can clear old content and insert the selected component:

~~~ts
import { Component, Type, ViewChild } from '@angular/core';
import { WidgetHostDirective } from './widget-host.directive';

@Component({
  selector: 'app-widget-panel',
  template: '<ng-container appWidgetHost />'
})
export class WidgetPanelComponent {
  @ViewChild(WidgetHostDirective)
  private host!: WidgetHostDirective;

  show<T>(componentType: Type<T>): void {
    const container = this.host.viewContainer;
    container.clear();
    container.createComponent(componentType);
  }
}
~~~

Clearing a view container destroys the views it owns. It is useful when one widget should replace another. If multiple components should remain, do not clear before every insertion.

## 4. Set inputs and handle outputs

A ComponentRef represents a dynamically created component. setInput is the supported way to assign a dynamic component input because Angular can schedule change detection and input lifecycle behavior appropriately.

~~~ts
const ref = container.createComponent(NoticeComponent);
ref.setInput('message', 'Changes saved.');
ref.setInput('tone', 'success');

const subscription = ref.instance.dismissed.subscribe(() => {
  ref.destroy();
});
~~~

Keep any subscriptions you create under an explicit owner and unsubscribe when that owner or component is destroyed. Destroying a ComponentRef tears down the dynamic component and its view. Do not keep references to a destroyed component.

Newer Angular releases also support binding helpers on programmatic rendering APIs. Check the API reference for the installed release before using helpers such as inputBinding or outputBinding.

## 5. Lazy-load a known component only when needed

If a feature is known but expensive to download, prefer @defer when it meets the interaction requirement. When the component type itself must be chosen at runtime, a normal dynamic import can load a known module:

~~~ts
async loadAdvancedSettings(): Promise<void> {
  const { AdvancedSettingsComponent } =
    await import('./advanced-settings/advanced-settings');
  this.show(AdvancedSettingsComponent);
}
~~~

The path above is a static import path in application code. Do not construct a path from user-provided strings. Handle import errors and show a useful fallback if the chunk fails to load.

## 6. Know which API owns the host element

ViewContainerRef.createComponent inserts the component at the view container and keeps it in Angular's view hierarchy. The standalone createComponent function returns a ComponentRef but does not automatically attach it to an existing container or application view. The caller must attach the host element, register the view for change detection, and remove and destroy it correctly.

Use the more explicit standalone API only when the insertion location is outside the current view hierarchy and there is a clear lifecycle owner, such as a carefully implemented overlay service.

## 7. Avoid unsafe and unbounded dynamic rendering

Dynamic rendering expands what the application can instantiate. Keep a finite registry and validate choices. Do not let remote data specify arbitrary JavaScript modules or components. Avoid creating many components without removing or reusing them; unbounded creation can retain memory and event listeners.

Use a clear owner for each ComponentRef. Define when it is replaced, removed, and destroyed. Handle render errors at an appropriate boundary. Current APIs may offer an onError option for rendering and change-detection errors; verify its availability and stability for the Angular release in use.

## 8. Consider server rendering and hydration

Programmatically inserting raw DOM nodes outside Angular can cause the client-rendered DOM to differ from the server-rendered tree. Prefer Angular's rendering APIs, and verify the behavior in SSR and hydration if the component appears on an initially server-rendered route. Do not treat hydration warnings as cosmetic; mismatched ownership can cause missing or duplicated content.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="099-reusable-ui-components-and-design-system-boundaries.md">099 - Reusable UI Components and Design-System Boundaries</a></td><td><a href="../qnas/100-dynamic-components-and-programmatic-rendering.md">Q&amp;A</a></td><td><a href="../examples/100-dynamic-components-and-programmatic-rendering.md">Example</a></td><td><a href="101-global-error-handling-logging-and-diagnostics.md">101 - Global Error Handling, Logging, and Diagnostics</a></td>
</tr>
  </tbody>
</table>
