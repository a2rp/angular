# Example: Provider recipes, multi providers, and provider functions

This standalone Angular example combines the provider recipes from the lesson in one small text-preview app:

- **useValue** supplies a typed application configuration object.
- **useFactory** derives a build label from that configuration.
- **useClass** selects the clock implementation for a clock token.
- **useExisting** exposes one logger instance through a separate token.
- **multi: true** collects independent text transforms into an array.
- **provideTextPipeline()** and its **with...()** options expose a typed environment setup API.

The files below fit inside a standalone Angular CLI application. The example uses current standalone APIs and Angular template control flow.

## 1. Define the tokens and contracts

Create **src/app/app.tokens.ts**:

~~~ts
import { InjectionToken } from '@angular/core';

export interface AppConfig {
  readonly name: string;
}

export const APP_CONFIG = new InjectionToken<AppConfig>('APP_CONFIG');

export const APP_BUILD_LABEL =
  new InjectionToken<string>('APP_BUILD_LABEL');

export interface Clock {
  now(): Date;
}

export const APP_CLOCK = new InjectionToken<Clock>('APP_CLOCK');

export interface AppLogger {
  log(message: string): void;
}

export const APP_LOGGER = new InjectionToken<AppLogger>('APP_LOGGER');

export interface TextTransform {
  readonly name: string;
  transform(text: string): string;
}

export const TEXT_TRANSFORMS =
  new InjectionToken<readonly TextTransform[]>('TEXT_TRANSFORMS');
~~~

A token's type describes the value consumers receive. In particular, **TEXT_TRANSFORMS** is typed as a readonly array because a multi provider injects the collected contributions as an array.

## 2. Implement the concrete services

Create **src/app/app.services.ts**:

~~~ts
import { Injectable } from '@angular/core';
import type { AppLogger, Clock } from './app.tokens';

@Injectable()
export class SystemClock implements Clock {
  now(): Date {
    return new Date();
  }
}

@Injectable()
export class ConsoleLogger implements AppLogger {
  log(message: string): void {
    console.info('[provider demo]', message);
  }
}
~~~

The example registers **SystemClock** under the **APP_CLOCK** token with **useClass**. It registers **ConsoleLogger** under its class token, then aliases **APP_LOGGER** to that class token with **useExisting**. Both injections therefore refer to the same object.

## 3. Add a feature-based text pipeline API

Create **src/app/text-pipeline.providers.ts**:

~~~ts
import { makeEnvironmentProviders } from '@angular/core';
import type { EnvironmentProviders, Provider } from '@angular/core';
import { TEXT_TRANSFORMS } from './app.tokens';
import type { TextTransform } from './app.tokens';

export interface TextPipelineFeature {
  readonly providers: readonly Provider[];
}

function transformFeature(transform: TextTransform): TextPipelineFeature {
  return {
    providers: [
      {
        provide: TEXT_TRANSFORMS,
        useValue: transform,
        multi: true,
      },
    ],
  };
}

export function withPrefix(prefixText: string): TextPipelineFeature {
  return transformFeature({
    name: 'Prefix',
    transform: (text) => prefixText + text,
  });
}

export function withUppercase(): TextPipelineFeature {
  return transformFeature({
    name: 'Uppercase',
    transform: (text) => text.toUpperCase(),
  });
}

export function withSuffix(suffixText: string): TextPipelineFeature {
  return transformFeature({
    name: 'Suffix',
    transform: (text) => text + suffixText,
  });
}

export function provideTextPipeline(
  ...features: readonly TextPipelineFeature[]
): EnvironmentProviders {
  const providers = features.flatMap((feature) => [...feature.providers]);

  return makeEnvironmentProviders(providers);
}
~~~

Each **with...()** function returns a small typed feature object. **provideTextPipeline()** gathers those feature providers and wraps them with **makeEnvironmentProviders()**. This communicates that the providers belong in an environment injector, such as application configuration.

The feature options each add one value to **TEXT_TRANSFORMS**. The pipeline does not need to know which options the application selected.

## 4. Consume the multi-provider array

Create **src/app/text-pipeline.service.ts**:

~~~ts
import { Injectable, inject } from '@angular/core';
import { TEXT_TRANSFORMS } from './app.tokens';

export interface TransformResult {
  readonly name: string;
  readonly output: string;
}

@Injectable({ providedIn: 'root' })
export class TextPipelineService {
  private readonly transforms = inject(TEXT_TRANSFORMS, {
    optional: true,
  }) ?? [];

  preview(input: string): TransformResult[] {
    return this.transforms.map((transform) => ({
      name: transform.name,
      output: transform.transform(input),
    }));
  }
}
~~~

The optional lookup lets this service still be created if the application omits **provideTextPipeline()**. In that case it has no transforms and returns an empty result list. The return type is an array because this service maps the injected multi-provider array into preview rows.

## 5. Configure the application providers

Create **src/app/app.config.ts**:

~~~ts
import { inject } from '@angular/core';
import type { ApplicationConfig } from '@angular/core';
import {
  APP_BUILD_LABEL,
  APP_CLOCK,
  APP_CONFIG,
  APP_LOGGER,
} from './app.tokens';
import type { AppConfig } from './app.tokens';
import { ConsoleLogger, SystemClock } from './app.services';
import {
  provideTextPipeline,
  withPrefix,
  withSuffix,
  withUppercase,
} from './text-pipeline.providers';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: APP_CONFIG,
      useValue: {
        name: 'Provider recipes demo',
      } satisfies AppConfig,
    },
    {
      provide: APP_BUILD_LABEL,
      useFactory: (config: AppConfig) => config.name + ' is ready',
      deps: [APP_CONFIG],
    },
    {
      provide: APP_CLOCK,
      useClass: SystemClock,
    },
    ConsoleLogger,
    {
      provide: APP_LOGGER,
      useExisting: ConsoleLogger,
    },
    provideTextPipeline(
      withPrefix('Angular says: '),
      withUppercase(),
      withSuffix(' - ready'),
    ),
  ],
};
~~~

The factory receives **APP_CONFIG** through its **deps** list, so Angular supplies the dependency when it creates **APP_BUILD_LABEL**. A factory can also call **inject()** when Angular invokes it in an injection context; this example uses **deps** to make the input explicit.

**useExisting** aliases **APP_LOGGER** to the instance already registered as **ConsoleLogger**. Replacing it with **useClass: ConsoleLogger** would create a separate provider record and could produce a second logger instance.

The **satisfies AppConfig** check keeps the supplied object aligned with the interface while preserving its inferred type. If the TypeScript version configured by your Angular project does not support **satisfies**, remove that keyword and keep the object shape unchanged.

## 6. Build a small standalone UI

Create **src/app/app.component.ts**:

~~~ts
import {
  Component,
  computed,
  inject,
  signal,
} from '@angular/core';
import {
  APP_BUILD_LABEL,
  APP_CLOCK,
  APP_CONFIG,
  APP_LOGGER,
} from './app.tokens';
import { ConsoleLogger } from './app.services';
import { TextPipelineService } from './text-pipeline.service';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
})
export class AppComponent {
  private readonly config = inject(APP_CONFIG);
  private readonly clock = inject(APP_CLOCK);
  private readonly logger = inject(APP_LOGGER);
  private readonly concreteLogger = inject(ConsoleLogger);
  private readonly pipeline = inject(TextPipelineService);

  readonly buildLabel = inject(APP_BUILD_LABEL);
  readonly appName = this.config.name;
  readonly createdAt = this.clock.now().toLocaleTimeString();
  readonly isLoggerAlias = this.logger === this.concreteLogger;
  readonly text = signal('Hello providers');
  readonly results = computed(() => this.pipeline.preview(this.text()));

  updateText(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.text.set(input.value);
  }

  logPreview(): void {
    this.logger.log('Preview requested for: ' + this.text());
  }
}
~~~

Create **src/app/app.component.html**:

~~~html
<main>
  <h1>{{ appName }}</h1>
  <p>{{ buildLabel }}</p>
  <p>Application clock read at: {{ createdAt }}</p>

  <label for="preview-text">Text to preview</label>
  <input
    id="preview-text"
    type="text"
    [value]="text()"
    (input)="updateText($event)"
  />

  <button type="button" (click)="logPreview()">Log current input</button>

  <h2>Configured transforms</h2>
  @if (results().length > 0) {
    <ul>
      @for (result of results(); track $index) {
        <li><strong>{{ result.name }}:</strong> {{ result.output }}</li>
      }
    </ul>
  } @else {
    <p>No transforms are configured.</p>
  }

  <p>
    APP_LOGGER and ConsoleLogger share one instance:
    <strong>{{ isLoggerAlias }}</strong>
  </p>
</main>
~~~

The **isLoggerAlias** value makes the alias relationship visible in the UI. The button also exercises the logger provided through the abstraction token.

Create **src/main.ts**:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error(error);
});
~~~

## 7. What to expect

Open the app. It should show the configured application name and build label, followed by three transform previews for the default text:

- **Prefix:** Angular says: Hello providers
- **Uppercase:** HELLO PROVIDERS
- **Suffix:** Hello providers - ready

Edit the text field and each preview updates. Click **Log current input** to see the logger message in the browser console. The alias indicator should be **true** because both tokens resolve to the same **ConsoleLogger** instance.

The results are shown separately so each registered transform is easy to inspect. This UI does not rely on one transform's output becoming the next transform's input.

## 8. Change the selected features

The feature API is configured in one place. For example, omit the uppercase option:

~~~ts
provideTextPipeline(
  withPrefix('Angular says: '),
  withSuffix(' - ready'),
)
~~~

Or use a different prefix and add another feature function:

~~~ts
provideTextPipeline(
  withPrefix('Welcome, '),
  withUppercase(),
)
~~~

Keep the feature API focused. Add options when they represent real capabilities consumers need, and validate mutually exclusive options rather than silently accepting contradictory configuration.

## 9. Provider recipe checklist

| Requirement | Configuration in this example |
| --- | --- |
| Supply an existing config object | APP_CONFIG with useValue |
| Derive a value from another token | APP_BUILD_LABEL with useFactory and deps |
| Choose the class for an abstraction token | APP_CLOCK with useClass |
| Alias another token to one existing class instance | APP_LOGGER with useExisting |
| Collect optional transform contributions | TEXT_TRANSFORMS with multi: true |
| Provide a safe environment-scoped setup API | provideTextPipeline() with makeEnvironmentProviders() |

Register one token consistently: do not mix a regular provider and multi providers for the same token in one injector. Also, multi providers collect contributions within their injector; do not assume values automatically merge across parent and child injector scopes.

## Related files

- [Lesson: Provider recipes, multi providers, and provider functions](../lessons/055-provider-recipes-multi-providers-and-provider-functions.md)
- [Q&A: Provider recipes, multi providers, and provider functions](../qnas/055-provider-recipes-multi-providers-and-provider-functions.md)
- [Next lesson: Injection contexts and resource lifetimes](../lessons/056-injection-contexts-and-resource-lifetimes.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="054-injectiontoken-and-typed-application-configuration.md">054 - InjectionToken and typed application configuration</a></td><td><a href="../lessons/055-provider-recipes-multi-providers-and-provider-functions.md">Lesson</a></td><td><a href="../qnas/055-provider-recipes-multi-providers-and-provider-functions.md">Q&amp;A</a></td><td><a href="056-injection-contexts-and-resource-lifetimes.md">056 - Injection contexts and resource lifetimes</a></td>
</tr>
  </tbody>
</table>
