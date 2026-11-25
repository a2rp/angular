# Examples: Two-Way Binding and ngModel

This walkthrough builds a small template-driven profile editor. It demonstrates two-way value synchronization, control registration, validation state, a standalone control, form submission, and programmatic model updates.

## 1. Create the component class

Create src/app/profile-editor.ts:

~~~ts
import { Component } from '@angular/core';
import { FormsModule, NgForm } from '@angular/forms';

interface Profile {
  displayName: string;
  email: string;
  receiveUpdates: boolean;
  favoriteColor: string;
}

@Component({
  selector: 'app-profile-editor',
  standalone: true,
  imports: [FormsModule],
  templateUrl: './profile-editor.html',
  styleUrl: './profile-editor.css',
})
export class ProfileEditor {
  profile: Profile = {
    displayName: 'Ada Lovelace',
    email: 'ada@example.test',
    receiveUpdates: true,
    favoriteColor: 'violet',
  };

  showHints = false;
  savedMessage = 'Update the fields, then save your profile.';

  saveProfile(form: NgForm): void {
    if (form.invalid) {
      form.control.markAllAsTouched();
      this.savedMessage = 'Correct the invalid fields before saving.';
      return;
    }

    this.savedMessage =
      'Saved profile for ' + form.value.displayName + '.';
  }

  loadExampleValues(): void {
    this.profile = {
      displayName: 'Grace Hopper',
      email: 'grace@example.test',
      receiveUpdates: false,
      favoriteColor: 'blue',
    };

    this.savedMessage = 'Example values loaded into the model.';
  }
}
~~~

FormsModule provides the ngModel and ngForm directives. The component property stores the profile data. NgForm, supplied to saveProfile, represents the registered form and its aggregate state.

## 2. Add the template

Create src/app/profile-editor.html:

~~~html
<section aria-labelledby="profile-title">
  <h2 id="profile-title">Profile editor</h2>

  <form
    #profileForm="ngForm"
    (ngSubmit)="saveProfile(profileForm)"
    novalidate
  >
    <div class="field">
      <label for="display-name">Display name</label>
      <input
        id="display-name"
        name="displayName"
        type="text"
        [(ngModel)]="profile.displayName"
        required
        minlength="2"
        #displayNameControl="ngModel"
        aria-describedby="display-name-help"
      >
      <small id="display-name-help">Enter at least two characters.</small>
      <p>
        Name control: {{ displayNameControl.valid ? 'valid' : 'invalid' }},
        {{ displayNameControl.dirty ? 'changed' : 'not changed' }},
        {{ displayNameControl.touched ? 'touched' : 'untouched' }}.
      </p>
    </div>

    <div class="field">
      <label for="email">Email</label>
      <input
        id="email"
        name="email"
        type="email"
        [(ngModel)]="profile.email"
        required
        email
        #emailControl="ngModel"
        aria-describedby="email-help"
      >
      <small id="email-help">Use a valid email address.</small>
      <p>Email control: {{ emailControl.valid ? 'valid' : 'invalid' }}.</p>
    </div>

    <div class="field">
      <label for="favorite-color">Favorite color</label>
      <select
        id="favorite-color"
        name="favoriteColor"
        [(ngModel)]="profile.favoriteColor"
      >
        <option value="violet">Violet</option>
        <option value="blue">Blue</option>
        <option value="green">Green</option>
      </select>
    </div>

    <label class="check-field">
      <input
        type="checkbox"
        name="receiveUpdates"
        [(ngModel)]="profile.receiveUpdates"
      >
      Receive updates
    </label>

    <label class="check-field">
      <input
        type="checkbox"
        [(ngModel)]="showHints"
        [ngModelOptions]="{ standalone: true }"
      >
      Show extra hints (not part of the profile form)
    </label>

    <p>Form valid: {{ profileForm.valid }}</p>
    <p>Submitted: {{ profileForm.submitted }}</p>

    <div class="actions">
      <button type="submit">Save profile</button>
      <button type="button" (click)="loadExampleValues()">
        Load example values
      </button>
    </div>
  </form>

  <p role="status">{{ savedMessage }}</p>
  <p>Live preview: {{ profile.displayName }} — {{ profile.email }}</p>
  <p>Receive updates: {{ profile.receiveUpdates ? 'Yes' : 'No' }}</p>
  <p>Show hints: {{ showHints ? 'Yes' : 'No' }}</p>
</section>
~~~

Each profile field inside the form has a name, so NgForm can register it. The showHints checkbox uses standalone ngModel options; it still synchronizes with the component but is not registered in profileForm.

The template reference variables export the NgForm and NgModel directives. They expose aggregate form state and individual control state without querying the DOM.

## 3. Add component styles

Create src/app/profile-editor.css:

~~~css
:host {
  display: block;
  max-width: 42rem;
  color: #1f2937;
}

section {
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #fff;
}

.field {
  display: grid;
  gap: 0.35rem;
  margin-block: 1rem;
}

label {
  font-weight: 600;
}

input[type="text"],
input[type="email"],
select {
  box-sizing: border-box;
  width: 100%;
  padding: 0.6rem;
  font: inherit;
}

input.ng-invalid.ng-touched {
  border: 2px solid #b91c1c;
}

small {
  color: #475569;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-block: 1rem;
}

button {
  padding: 0.6rem 0.9rem;
  font: inherit;
  cursor: pointer;
}
~~~

Angular adds status classes such as ng-invalid, ng-valid, ng-dirty, ng-pristine, ng-touched, and ng-untouched to controls managed by ngModel. The CSS highlights an invalid field after it has been touched.

## 4. Render the component

Import ProfileEditor into the root component and place its selector in the root template. For example, if the root component is src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { ProfileEditor } from './profile-editor';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ProfileEditor],
  template: '<app-profile-editor />',
})
export class App {}
~~~

If the root component already has a template, imports, or providers, preserve them and add ProfileEditor and app-profile-editor.

Start the app:

~~~sh
ng serve
~~~

## 5. Try the interactions

1. Edit the display name. The preview updates because profile.displayName is two-way bound.
2. Clear the email or enter an invalid address. Its control and the form become invalid.
3. Move away from an edited field. Its touched state changes.
4. Submit an invalid form. saveProfile marks the controls touched and keeps the user on the page.
5. Correct the values and submit again. The form is valid and the status message confirms the saved name.
6. Toggle Receive updates. Its boolean value appears in the preview and is part of the form.
7. Toggle Show extra hints. Its component value changes, but it is excluded from the form's registered value.
8. Activate Load example values. The component model changes programmatically and the input controls reflect those values.

## 6. See the two-way binding expansion

For a simple control, this:

~~~html
<input name="displayName" [(ngModel)]="profile.displayName">
~~~

can be written as property and event bindings:

~~~html
<input
  name="displayName"
  [ngModel]="profile.displayName"
  (ngModelChange)="profile.displayName = $event"
>
~~~

The shorthand is clearer when no custom change handling is needed. If the application must normalize or inspect values, replace the event assignment with a method that updates the model intentionally.

## 7. Practice variations

1. Add a required phone field with a unique name and show its NgModel valid and touched states.
2. Add an ngModelOptions updateOn blur setting to a field and observe when the preview changes.
3. Add another presentation-only checkbox with ngModelOptions standalone and confirm it stays out of the form's registered data.
4. Remove a name from a registered control and observe the form registration problem; restore the meaningful name afterward.
5. Add an object-valued select option using Angular's ngValue binding.
6. Compare the template-driven ngModel approach with the reactive forms approach in its later lesson; do not put both directives on the same control.

Do not treat form synchronization as validation or server-side trust. Validate data according to application rules before using or storing it.
