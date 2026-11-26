# Example: FormArray and dynamic form structures

Build a profile editor with a changing list of typed skill rows. Each row has a required name and a level, and the collection requires at least one row. The example demonstrates a **FormArray** of consistently shaped **FormGroup** controls, template binding by index, row and collection validation, structural edits, reset behavior, and disabled-control values.

Use an Angular CLI application with standalone components and reactive forms. Replace the root component files or adapt the same model and template to another component.

## 1. Create the typed form model

Replace the root component TypeScript file:

~~~ts
import { JsonPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import {
  AbstractControl,
  FormArray,
  FormBuilder,
  FormControl,
  FormGroup,
  isFormArray,
  ReactiveFormsModule,
  ValidationErrors,
  ValidatorFn,
  Validators,
} from '@angular/forms';

type SkillLevel = 'beginner' | 'intermediate' | 'advanced';

interface SkillValue {
  name: string;
  level: SkillLevel;
}

interface ProfileValue {
  fullName: string;
  skills: SkillValue[];
}

type SkillControls = {
  name: FormControl<string>;
  level: FormControl<SkillLevel>;
};

type SkillGroup = FormGroup<SkillControls>;

const requireOneSkill: ValidatorFn = (
  control: AbstractControl,
): ValidationErrors | null => {
  if (!isFormArray(control)) {
    return null;
  }

  return control.length > 0
    ? null
    : { minItems: { required: 1, actual: 0 } };
};

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [JsonPipe, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly fb = inject(FormBuilder).nonNullable;

  readonly profileForm = this.fb.group({
    fullName: this.fb.control('', [
      Validators.required,
      Validators.minLength(2),
      Validators.maxLength(60),
    ]),
    skills: this.fb.array<SkillGroup>([this.createSkill()], {
      validators: requireOneSkill,
    }),
  });

  attemptedSubmit = false;
  savedProfile: ProfileValue | null = null;

  get skills(): FormArray<SkillGroup> {
    return this.profileForm.controls.skills;
  }

  private createSkill(value: SkillValue = {
    name: '',
    level: 'beginner',
  }): SkillGroup {
    return this.fb.group({
      name: this.fb.control(value.name, [
        Validators.required,
        Validators.maxLength(40),
      ]),
      level: this.fb.control<SkillLevel>(value.level),
    });
  }

  addSkill(): void {
    this.skills.push(this.createSkill());
    this.skills.markAsDirty();
    this.savedProfile = null;
  }

  removeSkill(index: number): void {
    if (index < 0 || index >= this.skills.length) {
      return;
    }

    this.skills.removeAt(index);
    this.skills.markAsDirty();
    this.savedProfile = null;
  }

  toggleFirstSkillDisabled(): void {
    const firstSkill = this.skills.at(0);

    if (!firstSkill) {
      return;
    }

    if (firstSkill.disabled) {
      firstSkill.enable();
    } else {
      firstSkill.disable();
    }

    this.savedProfile = null;
  }

  loadSample(): void {
    const sampleSkills: SkillValue[] = [
      { name: 'TypeScript', level: 'intermediate' },
      { name: 'Angular', level: 'beginner' },
    ];

    this.skills.clear({ emitEvent: false });

    for (const skill of sampleSkills) {
      this.skills.push(this.createSkill(skill), { emitEvent: false });
    }

    this.profileForm.setValue({
      fullName: 'Asha Rao',
      skills: sampleSkills,
    });

    this.profileForm.markAsPristine();
    this.profileForm.markAsUntouched();
    this.attemptedSubmit = false;
    this.savedProfile = null;
  }

  save(): void {
    this.attemptedSubmit = true;
    this.savedProfile = null;

    if (this.profileForm.invalid || this.profileForm.pending) {
      this.profileForm.markAllAsTouched();
      return;
    }

    this.savedProfile = this.profileForm.getRawValue();
    this.profileForm.markAsPristine();
  }

  reset(): void {
    this.profileForm.reset();

    // reset() resets existing controls but keeps them in the array.
    // Restore this example's initial one-row structure explicitly.
    this.skills.clear({ emitEvent: false });
    this.skills.push(this.createSkill(), { emitEvent: false });

    this.profileForm.markAsPristine();
    this.profileForm.markAsUntouched();
    this.attemptedSubmit = false;
    this.savedProfile = null;
  }
}
~~~

The form uses a non-nullable builder, so the name and level controls have string and **SkillLevel** values rather than nullable values. The **SkillGroup** type makes every array row use the same control shape.

The **createSkill()** method is the only place that defines the row structure and its validation rules. **addSkill()** and **loadSample()** reuse it instead of constructing slightly different groups in multiple places. The **requireOneSkill** validator belongs to the collection and makes an empty array invalid.

## 2. Bind the form and repeated rows

Replace the root component template:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Reactive forms</p>
    <h1>Profile skills</h1>
    <p>
      Add, edit, and remove skill rows. Each row is a typed group inside a
      dynamic form array.
    </p>
  </header>

  <form [formGroup]="profileForm" (ngSubmit)="save()" novalidate>
    <div class="field">
      <label for="full-name">
        Full name <span class="required">(required)</span>
      </label>
      <input
        id="full-name"
        type="text"
        formControlName="fullName"
        autocomplete="name"
        aria-describedby="full-name-help"
        [attr.aria-invalid]="
          profileForm.controls.fullName.invalid &&
          (profileForm.controls.fullName.touched || attemptedSubmit)
            ? 'true'
            : null
        "
      />
      <p class="hint" id="full-name-help">
        Enter between 2 and 60 characters.
      </p>

      @if (
        profileForm.controls.fullName.invalid &&
        (profileForm.controls.fullName.touched || attemptedSubmit)
      ) {
        <p class="error" role="alert">
          @if (profileForm.controls.fullName.hasError('required')) {
            Enter your name.
          } @else if (profileForm.controls.fullName.hasError('minlength')) {
            Use at least two characters.
          } @else if (profileForm.controls.fullName.hasError('maxlength')) {
            Use no more than 60 characters.
          }
        </p>
      }
    </div>

    <fieldset class="skills" formArrayName="skills">
      <legend>Skills</legend>
      <p class="hint">
        Add at least one skill. Each row has a name and a proficiency level.
      </p>

      @for (skillGroup of skills.controls; track skillGroup; let i = $index) {
        <fieldset class="skill-row" [formGroupName]="i">
          <legend>Skill {{ i + 1 }}</legend>

          <div class="field">
            <label [for]="'skill-name-' + i">
              Skill name <span class="required">(required)</span>
            </label>
            <input
              [id]="'skill-name-' + i"
              type="text"
              formControlName="name"
              autocomplete="off"
              [attr.aria-invalid]="
                skillGroup.controls.name.invalid &&
                (skillGroup.controls.name.touched || attemptedSubmit)
                  ? 'true'
                  : null
              "
            />

            @if (
              skillGroup.controls.name.invalid &&
              (skillGroup.controls.name.touched || attemptedSubmit)
            ) {
              <p class="error" [id]="'skill-name-error-' + i" role="alert">
                @if (skillGroup.controls.name.hasError('required')) {
                  Enter a skill name.
                } @else if (skillGroup.controls.name.hasError('maxlength')) {
                  Use no more than 40 characters.
                }
              </p>
            }
          </div>

          <div class="field">
            <label [for]="'skill-level-' + i">Level</label>
            <select [id]="'skill-level-' + i" formControlName="level">
              <option value="beginner">Beginner</option>
              <option value="intermediate">Intermediate</option>
              <option value="advanced">Advanced</option>
            </select>
          </div>

          <button
            class="button button-secondary remove-button"
            type="button"
            [attr.aria-label]="'Remove skill ' + (i + 1)"
            (click)="removeSkill(i)"
          >
            Remove skill
          </button>
        </fieldset>
      } @empty {
        <p class="empty-state">
          No skill rows yet. Choose “Add skill” to create one.
        </p>
      }

      @if (skills.hasError('minItems') && (skills.touched || attemptedSubmit)) {
        <p class="error" role="alert">Add at least one skill before saving.</p>
      }
    </fieldset>

    <div class="actions">
      <button
        class="button"
        type="button"
        (click)="addSkill()"
      >
        Add skill
      </button>

      <button
        class="button button-secondary"
        type="button"
        (click)="toggleFirstSkillDisabled()"
        [disabled]="skills.length === 0"
      >
        Toggle first row disabled
      </button>

      <button
        class="button button-secondary"
        type="button"
        (click)="loadSample()"
      >
        Load sample data
      </button>

      <button
        class="button button-secondary"
        type="button"
        (click)="reset()"
      >
        Reset
      </button>

      <button class="button button-primary" type="submit">
        Save profile
      </button>
    </div>

    @if (attemptedSubmit && profileForm.invalid) {
      <p class="error-summary" role="alert">
        The profile was not saved. Correct the fields marked above.
      </p>
    }
  </form>

  <section class="diagnostics" aria-labelledby="diagnostics-title">
    <h2 id="diagnostics-title">Form values</h2>
    <p>
      The enabled value omits disabled controls. The raw value includes them.
    </p>

    <h3>profileForm.value</h3>
    <pre>{{ profileForm.value | json }}</pre>

    <h3>profileForm.getRawValue()</h3>
    <pre>{{ profileForm.getRawValue() | json }}</pre>

    <dl>
      <dt>Form status</dt>
      <dd>{{ profileForm.status }}</dd>

      <dt>Skills array length</dt>
      <dd>{{ skills.length }}</dd>

      <dt>Skills array state</dt>
      <dd>{{ skills.dirty ? 'dirty' : 'pristine' }}</dd>
    </dl>
  </section>

  @if (savedProfile; as profile) {
    <section class="success" aria-labelledby="saved-title" role="status">
      <h2 id="saved-title">Saved profile snapshot</h2>
      <p>{{ profile.fullName }}</p>

      @if (profile.skills.length > 0) {
        <ul>
          @for (skill of profile.skills; track $index) {
            <li>{{ skill.name }} - {{ skill.level }}</li>
          }
        </ul>
      } @else {
        <p>The saved profile has no skill rows.</p>
      }

      <p class="hint">
        This demonstration saves the raw form value, so disabled rows are
        included. Choose the payload deliberately in a real application.
      </p>
    </section>
  }
</main>
~~~

Every repeated control uses its current numeric index to connect to the matching group. The row itself is tracked by its control object. Each input has a label, and each remove button identifies which row it removes.

## 3. Add responsive styling

Replace the component stylesheet:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.page {
  box-sizing: border-box;
  width: min(100% - 2rem, 52rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 2rem;
}

.eyebrow {
  margin: 0 0 0.5rem;
  color: #4256a6;
  font-size: 0.875rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

h1,
h2,
h3 {
  line-height: 1.2;
}

h1 {
  margin: 0 0 0.75rem;
  font-size: clamp(2rem, 6vw, 3rem);
}

h2 {
  margin-top: 0;
  font-size: 1.35rem;
}

h3 {
  margin-bottom: 0.5rem;
  font-size: 1rem;
}

.page-header > p:last-child {
  max-width: 65ch;
  color: #475467;
  line-height: 1.6;
}

form {
  display: grid;
  gap: 1.25rem;
}

.field {
  display: grid;
  gap: 0.5rem;
}

label,
legend {
  font-weight: 650;
}

.required {
  color: #475467;
  font-size: 0.875rem;
  font-weight: 400;
}

input,
select {
  box-sizing: border-box;
  width: 100%;
  min-height: 2.75rem;
  padding: 0.65rem 0.75rem;
  border: 1px solid #98a2b3;
  border-radius: 0.4rem;
  background: #fff;
  color: inherit;
  font: inherit;
}

input:focus-visible,
select:focus-visible,
button:focus-visible {
  outline: 3px solid #84adff;
  outline-offset: 2px;
}

[aria-invalid='true'] {
  border-color: #b42318;
}

fieldset {
  min-width: 0;
  margin: 0;
  padding: 1rem;
  border: 1px solid #c5cbd5;
  border-radius: 0.65rem;
}

.skills {
  display: grid;
  gap: 1rem;
}

.skills > legend {
  padding-inline: 0.35rem;
  font-size: 1.15rem;
}

.skill-row {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr) auto;
  align-items: end;
  gap: 1rem;
  background: #f8f9fc;
}

.skill-row > legend {
  padding-inline: 0.35rem;
}

.hint,
.error,
.error-summary,
.empty-state {
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.5;
}

.hint {
  color: #475467;
}

.error,
.error-summary {
  color: #b42318;
}

.error-summary {
  font-weight: 650;
}

.empty-state {
  padding: 1rem;
  border: 1px dashed #98a2b3;
  border-radius: 0.5rem;
  color: #475467;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.button {
  min-height: 2.75rem;
  padding: 0.65rem 1rem;
  border: 1px solid #253b80;
  border-radius: 0.4rem;
  background: #253b80;
  color: #fff;
  cursor: pointer;
  font: inherit;
  font-weight: 650;
}

.button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.button-secondary {
  border-color: #98a2b3;
  background: #fff;
  color: #172033;
}

.button-primary {
  border-color: #176b3a;
  background: #176b3a;
}

.remove-button {
  white-space: nowrap;
}

.diagnostics,
.success {
  margin-top: 1.5rem;
  padding: 1.25rem;
  border: 1px solid #98a2b3;
  border-radius: 0.75rem;
}

.diagnostics > p {
  color: #475467;
  line-height: 1.5;
}

pre {
  max-width: 100%;
  padding: 1rem;
  overflow: auto;
  border-radius: 0.5rem;
  background: #172033;
  color: #f8fafc;
  line-height: 1.5;
}

dl {
  display: grid;
  grid-template-columns: minmax(10rem, 14rem) 1fr;
  gap: 0.5rem 1rem;
}

dt {
  font-weight: 700;
}

dd {
  min-width: 0;
  margin: 0;
  overflow-wrap: anywhere;
}

.success {
  border-color: #75c79a;
  background: #f0fdf4;
}

.success h2 {
  margin-top: 0;
}

@media (max-width: 42rem) {
  .page {
    width: min(100% - 1rem, 52rem);
    padding-block: 1.5rem;
  }

  .skill-row {
    grid-template-columns: 1fr;
  }

  .remove-button {
    justify-self: start;
  }

  dl {
    grid-template-columns: 1fr;
    gap: 0.2rem;
  }

  dd {
    margin-bottom: 0.6rem;
  }
}
~~~

## 4. Explore the behavior

1. Submit the empty form. The full-name and skill-name errors appear, and the form is not saved.
2. Enter a full name and a skill name. Choose a level and submit again. The saved snapshot shows the **getRawValue()** result.
3. Choose **Add skill**. A new row is registered in the **FormArray**, with the same controls and validators as the first row.
4. Add a second skill, then remove the first. Check that the remaining row still displays its own values and its label remains associated with its input.
5. Remove every row and submit. The collection-level **requireOneSkill** validator shows the error and blocks saving.
6. Choose **Load sample data**. The current rows are replaced with two sample rows, and the form is marked pristine.
7. Choose **Toggle first row disabled**. Compare **profileForm.value** with **profileForm.getRawValue()**. The disabled row is absent from the enabled aggregate value but remains in the raw value.
8. With the first sample row disabled, save the form. The saved snapshot includes that row because this example intentionally calls **getRawValue()**. In a real submission, decide whether disabled values belong in the payload.
9. Choose **Reset** after adding or removing rows. The values and interaction state reset, and the example rebuilds its original one-row structure.

The disabled-row control is included to show value aggregation, not as a recommendation to disable invalid data before submission. Disabled controls are excluded from validation. The server must still validate any data it receives.

## Further practice

- Add a remove-all action that calls **skills.clear()** and marks the array dirty.
- Add a row-level validator that checks two fields together.
- Add a button that inserts a blank row at a chosen index with **insert()**.
- Add an edit action that replaces one row with **setControl()**.
- Load an arbitrary list from a data source by creating one group per item before calling **setValue()**.
- Change the save handler to submit **profileForm.value** and compare the result with the raw-value version.
- Decide whether resetting the form should restore the initial list or preserve the current number of rows, then encode that policy explicitly.

## Related material

- Read the [lesson](../lessons/065-formarray-and-dynamic-form-structures.md).
- Review the [Q&A](../qnas/065-formarray-and-dynamic-form-structures.md).
- Continue with [Synchronous validators and validation state](../lessons/066-synchronous-validators-and-validation-state.md).
- Revisit [Typed reactive forms and non-nullable controls](../lessons/064-typed-reactive-forms-and-non-nullable-controls.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="064-typed-reactive-forms-and-non-nullable-controls.md">064 - Typed reactive forms and non-nullable controls</a></td><td><a href="../lessons/065-formarray-and-dynamic-form-structures.md">Lesson</a></td><td><a href="../qnas/065-formarray-and-dynamic-form-structures.md">Q&amp;A</a></td><td><a href="066-synchronous-validators-and-validation-state.md">066 - Synchronous validators and validation state</a></td>
</tr>
  </tbody>
</table>
