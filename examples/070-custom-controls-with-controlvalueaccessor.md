# Example: Custom controls with ControlValueAccessor

Build a reusable five-star rating component and use it as a field in a parent Reactive Form. The component implements **ControlValueAccessor**, so Angular can write values to it, receive user selections, mark it touched, and disable it.

The widget uses native buttons rather than custom radio roles. Each button has an accessible name and pressed state, and works with standard keyboard activation. A fieldset and legend provide the group context.

Use an Angular CLI application. Put both components in the root component TypeScript file for this demo, or split them into separate files.

## 1. Create the rating component

Add the imports, then add this standalone component:

~~~ts
import {
  ChangeDetectionStrategy,
  ChangeDetectorRef,
  Component,
  ElementRef,
  Input,
  forwardRef,
  inject,
} from '@angular/core';
import {
  ControlValueAccessor,
  FormBuilder,
  NG_VALUE_ACCESSOR,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-rating-control',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <fieldset
      [id]="idPrefix + '-group'"
      [disabled]="disabled"
      [attr.aria-required]="required ? 'true' : null"
      [attr.aria-invalid]="errorMessage ? 'true' : null"
      [attr.aria-describedby]="
        errorMessage
          ? idPrefix + '-help ' + idPrefix + '-error'
          : idPrefix + '-help'
      "
      (focusout)="handleFocusOut($event)"
    >
      <legend>
        {{ label }}
        @if (required) {
          <span>(required)</span>
        }
      </legend>

      <p [id]="idPrefix + '-help'">
        Choose a rating from one to five. Use Tab to reach each button;
        press Enter or Space to choose it.
      </p>

      <div class="ratings">
        @for (rating of ratings; track rating) {
          <button
            type="button"
            [attr.aria-label]="rating + (rating === 1 ? ' star' : ' stars')"
            [attr.aria-pressed]="value === rating"
            (click)="selectRating(rating)"
          >
            <span aria-hidden="true">★</span>
          </button>
        }
      </div>

      @if (errorMessage) {
        <p [id]="idPrefix + '-error'">{{ errorMessage }}</p>
      }
    </fieldset>
  `,
  styles: [`
    :host {
      display: block;
    }

    fieldset {
      border: 1px solid #777;
      padding: 1rem;
    }

    .ratings {
      display: flex;
      gap: 0.5rem;
    }

    button {
      min-width: 2.5rem;
      min-height: 2.5rem;
      border: 2px solid #555;
      border-radius: 0.25rem;
      background: white;
      color: #222;
      font-size: 1.5rem;
      cursor: pointer;
    }

    button[aria-pressed='true'] {
      border-width: 4px;
      font-weight: bold;
    }

    button:focus-visible {
      outline: 3px solid #075db8;
      outline-offset: 2px;
    }

    fieldset:disabled button {
      cursor: not-allowed;
      opacity: 0.6;
    }
  `],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => RatingControlComponent),
      multi: true,
    },
  ],
})
export class RatingControlComponent implements ControlValueAccessor {
  @Input() idPrefix = 'rating';
  @Input() label = 'Rating';
  @Input() required = false;
  @Input() errorMessage: string | null = null;

  readonly ratings = [1, 2, 3, 4, 5];

  value: number | null = null;
  disabled = false;

  private onChange: (value: number | null) => void = () => {};
  private onTouched: () => void = () => {};

  private readonly host = inject(ElementRef<HTMLElement>);
  private readonly changeDetector = inject(ChangeDetectorRef);

  writeValue(value: number | null): void {
    // Model -> view. Do not call onChange from this method.
    this.value = value;
    this.changeDetector.markForCheck();
  }

  registerOnChange(fn: (value: number | null) => void): void {
    this.onChange = fn;
  }

  registerOnTouched(fn: () => void): void {
    this.onTouched = fn;
  }

  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
    this.changeDetector.markForCheck();
  }

  selectRating(rating: number): void {
    if (this.disabled || this.value === rating) {
      return;
    }

    // View -> model. This is a real user selection.
    this.value = rating;
    this.onChange(rating);
  }

  handleFocusOut(event: FocusEvent): void {
    const nextTarget = event.relatedTarget as Node | null;

    // Moving between this widget's buttons does not leave the form control.
    if (nextTarget && this.host.nativeElement.contains(nextTarget)) {
      return;
    }

    this.onTouched();
  }
}
~~~

The **idPrefix** input must be unique for each rating component instance on the page. It keeps the fieldset, help, and error IDs unique and correctly associated.

## 2. Use the accessor in a parent form

Add a parent component in the same file:

~~~ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ReactiveFormsModule, RatingControlComponent],
  template: `
    <main>
      <h1>Product feedback</h1>
      <p id="form-help">Choose a topic and rate your experience.</p>

      <form [formGroup]="feedbackForm" (ngSubmit)="submit()" novalidate>
        @if (submitted && feedbackForm.invalid) {
          <section aria-labelledby="error-summary-title" role="alert">
            <h2 id="error-summary-title">Please review your feedback</h2>
            <ul>
              @if (feedbackForm.controls.topic.invalid) {
                <li><a href="#topic">Choose a topic.</a></li>
              }
              @if (feedbackForm.controls.rating.invalid) {
                <li><a href="#product-rating-group">Choose a rating.</a></li>
              }
            </ul>
          </section>
        }

        <div>
          <label for="topic">Topic (required)</label>
          <p id="topic-help">Choose the part of the product you are reviewing.</p>
          <select
            id="topic"
            formControlName="topic"
            required
            [attr.aria-describedby]="
              showTopicError ? 'topic-help topic-error' : 'topic-help'
            "
            [attr.aria-invalid]="showTopicError ? 'true' : null"
          >
            <option value="">Choose a topic</option>
            <option value="setup">Setup</option>
            <option value="performance">Performance</option>
            <option value="support">Support</option>
          </select>
          @if (showTopicError) {
            <p id="topic-error">Choose a topic.</p>
          }
        </div>

        <app-rating-control
          formControlName="rating"
          idPrefix="product-rating"
          label="How would you rate the product?"
          [required]="true"
          [errorMessage]="ratingError"
        ></app-rating-control>

        <div>
          <label for="comments">Additional comments (optional)</label>
          <p id="comments-help">
            Do not include passwords or other sensitive information.
          </p>
          <textarea
            id="comments"
            formControlName="comments"
            rows="4"
            aria-describedby="comments-help"
          ></textarea>
        </div>

        <button type="submit">Submit feedback</button>
        <button type="button" (click)="resetForm()">Clear form</button>
      </form>

      <p role="status" aria-live="polite">{{ successMessage }}</p>
    </main>
  `,
})
export class AppComponent {
  private readonly fb = inject(FormBuilder);

  readonly feedbackForm = this.fb.group({
    topic: this.fb.nonNullable.control('', Validators.required),
    rating: this.fb.control<number | null>(null, Validators.required),
    comments: this.fb.nonNullable.control(''),
  });

  submitted = false;
  successMessage = '';

  get showTopicError(): boolean {
    const control = this.feedbackForm.controls.topic;
    return control.invalid && (control.touched || this.submitted);
  }

  get ratingError(): string | null {
    const control = this.feedbackForm.controls.rating;

    return control.invalid && (control.touched || this.submitted)
      ? 'Choose a rating between one and five stars.'
      : null;
  }

  submit(): void {
    this.submitted = true;
    this.feedbackForm.markAllAsTouched();

    if (this.feedbackForm.invalid || this.feedbackForm.pending) {
      return;
    }

    // This example demonstrates form integration; it sends no request.
    this.successMessage = 'Feedback is valid. No data was sent.';
  }

  resetForm(): void {
    this.feedbackForm.reset();
    this.submitted = false;
    this.successMessage = '';
  }
}
~~~

The parent applies **Validators.required** to the rating's **FormControl** and passes a visible error message into the custom component. CVA itself does not transfer the parent's error map into the component.

## 3. Try the integration

1. Submit without choosing a topic or rating. The parent form becomes touched and invalid, and its summary links to the missing fields.
2. Choose a topic. The topic's required error clears.
3. Tab through the five rating buttons. Each button has an accessible name and **aria-pressed** state; Enter or Space selects the focused rating.
4. Move focus from one rating button to another. The parent rating control should not become touched until focus leaves the whole widget.
5. Choose a rating. The widget calls **onChange**, and **feedbackForm.controls.rating.value** becomes the selected number.
6. Programmatically call **feedbackForm.controls.rating.setValue(4)**. The fourth button becomes selected through **writeValue()**.
7. Call **feedbackForm.controls.rating.disable()**. The fieldset and its buttons become disabled through **setDisabledState()**. Call **enable()** to make them interactive again.
8. Reset the parent form. Angular calls **writeValue(null)** and the stars clear without a user-change callback.
9. Submit a valid form. A polite status message appears; this demonstration does not transmit form data.

## 4. Keep the value directions straight

- **writeValue()** renders a value supplied by Angular.
- A user clicking a button updates the local view and calls **onChange()**.
- Focus leaving the whole rating widget calls **onTouched()**.
- A parent disable or enable operation calls **setDisabledState()**.
- The parent **FormControl** owns the value and required validator.
- The widget owns its internal accessible group, button names, and display of the error input.

If the rating control is reused more than once, pass a different **idPrefix** to each instance. In production, also style focus, selected, and disabled states to match the design system while keeping them visually distinct.

## Further reading

- [Custom controls with ControlValueAccessor](../lessons/070-custom-controls-with-controlvalueaccessor.md)
- [ControlValueAccessor API](https://angular.dev/api/forms/ControlValueAccessor)
- [NG_VALUE_ACCESSOR API](https://angular.dev/api/forms/NG_VALUE_ACCESSOR)
- [Angular reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [Angular form validation](https://angular.dev/guide/forms/form-validation)
