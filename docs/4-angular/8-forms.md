---
sidebar_position: 8
---

# Forms

Angular provides two approaches to handling user input through forms: **reactive forms** and **template-driven forms**. Angular 19 also introduces **signal forms** as a new experimental approach.

Source: [angular.dev/guide/forms](https://angular.dev/guide/forms)

## Choosing an Approach

| Feature | Reactive Forms | Template-Driven Forms |
|---|---|---|
| Form model | Explicit, created in the class | Implicit, created by directives |
| Data flow | Synchronous | Asynchronous |
| Validation | Functions in the class | Directives in the template |
| Scalability | Better for complex forms | Better for simple forms |
| Testing | Easier to unit test | Requires DOM testing |
| Module | `ReactiveFormsModule` | `FormsModule` |

## Reactive Forms

Reactive forms provide a model-driven approach. The form model is defined in the component class, giving you direct access to the form state.

### Setup

Import `ReactiveFormsModule` in your component:

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormControl, FormGroup, Validators } from '@angular/forms';

@Component({
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="profileForm" (ngSubmit)="onSubmit()">
      <label>
        First Name
        <input formControlName="firstName" />
      </label>
      <label>
        Last Name
        <input formControlName="lastName" />
      </label>
      <label>
        Email
        <input formControlName="email" type="email" />
      </label>
      <button type="submit" [disabled]="!profileForm.valid">Save</button>
    </form>
  `,
})
export class ProfileEditor {
  profileForm = new FormGroup({
    firstName: new FormControl('', Validators.required),
    lastName: new FormControl('', Validators.required),
    email: new FormControl('', [Validators.required, Validators.email]),
  });

  onSubmit() {
    console.log(this.profileForm.value);
  }
}
```

### FormControl

A `FormControl` represents a single input field. It tracks the value and validation status:

```typescript
const name = new FormControl('initial value');

console.log(name.value);    // 'initial value'
console.log(name.valid);    // true
console.log(name.dirty);    // false (user hasn't changed it)
console.log(name.touched);  // false (user hasn't focused it)
```

### FormGroup

A `FormGroup` aggregates multiple `FormControl` instances:

```typescript
const form = new FormGroup({
  username: new FormControl(''),
  password: new FormControl(''),
});

// Access child controls
form.get('username')?.value;

// Set all values at once
form.setValue({ username: 'admin', password: 'secret' });

// Patch partial values
form.patchValue({ username: 'admin' });
```

### FormArray

A `FormArray` manages a dynamic list of controls:

```typescript
const skills = new FormArray([
  new FormControl('Angular'),
  new FormControl('TypeScript'),
]);

// Add a control
skills.push(new FormControl('RxJS'));

// Remove a control
skills.removeAt(0);
```

```html
@for (skill of skills.controls; track $index) {
  <input [formControl]="skill" />
}
<button (click)="addSkill()">Add Skill</button>
```

### FormBuilder

`FormBuilder` provides a shorthand for creating form controls:

```typescript
import { FormBuilder, Validators } from '@angular/forms';

@Component({ ... })
export class ProfileEditor {
  private fb = inject(FormBuilder);

  profileForm = this.fb.group({
    firstName: ['', Validators.required],
    lastName: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
    address: this.fb.group({
      street: [''],
      city: [''],
      zip: [''],
    }),
  });
}
```

## Template-Driven Forms

Template-driven forms rely on directives in the template to create and manage the form model.

### Setup

Import `FormsModule` in your component:

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  imports: [FormsModule],
  template: `
    <form #profileForm="ngForm" (ngSubmit)="onSubmit(profileForm)">
      <label>
        Name
        <input name="name" [(ngModel)]="user.name" required />
      </label>
      <label>
        Email
        <input name="email" [(ngModel)]="user.email" required email />
      </label>
      <button type="submit" [disabled]="!profileForm.valid">Save</button>
    </form>
  `,
})
export class ProfileEditor {
  user = { name: '', email: '' };

  onSubmit(form: any) {
    console.log(form.value);
  }
}
```

### ngModel

`ngModel` creates a `FormControl` instance under the hood and binds it to the form element. Each control must have a `name` attribute.

## Signal Forms

Angular 19 introduces experimental signal-based forms that integrate with Angular's reactivity system:

```typescript
import { SignalFormBuilder } from '@angular/forms';

@Component({ ... })
export class ProfileEditor {
  private sfb = inject(SignalFormBuilder);

  profileForm = this.sfb.group({
    firstName: this.sfb.control('', { validators: Validators.required }),
    lastName: this.sfb.control(''),
  });

  // Access values as signals
  firstName = this.profileForm.controls.firstName.value;
}
```

Signal forms provide reactive access to form state through signals, allowing integration with `computed`, `effect`, and other signal-based APIs.

Source: [angular.dev/essentials/signal-forms](https://angular.dev/essentials/signal-forms)

## Validation

### Built-in Validators

Angular provides several built-in validators:

| Validator | Description |
|---|---|
| `Validators.required` | Value must not be empty |
| `Validators.email` | Value must be a valid email |
| `Validators.min(n)` | Numeric value must be ≥ n |
| `Validators.max(n)` | Numeric value must be ≤ n |
| `Validators.minLength(n)` | String length must be ≥ n |
| `Validators.maxLength(n)` | String length must be ≤ n |
| `Validators.pattern(regex)` | Value must match the regex |

### Custom Validators

Create custom validators as functions:

```typescript
import { AbstractControl, ValidationErrors } from '@angular/forms';

function forbiddenNameValidator(forbiddenName: string) {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = control.value === forbiddenName;
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  };
}

// Usage
new FormControl('', [Validators.required, forbiddenNameValidator('admin')]);
```

### Async Validators

For validation that requires server calls:

```typescript
function uniqueEmailValidator(userService: UserService) {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return userService.checkEmail(control.value).pipe(
      map(exists => exists ? { emailTaken: true } : null),
    );
  };
}
```

### Displaying Validation Errors

```html
<input formControlName="email" />
@if (profileForm.get('email')?.hasError('required') && profileForm.get('email')?.touched) {
  <p class="error">Email is required.</p>
}
@if (profileForm.get('email')?.hasError('email')) {
  <p class="error">Enter a valid email address.</p>
}
```

### Cross-Field Validation

Validate multiple fields together by adding a validator to the `FormGroup`:

```typescript
function passwordMatchValidator(group: AbstractControl): ValidationErrors | null {
  const password = group.get('password')?.value;
  const confirm = group.get('confirmPassword')?.value;
  return password === confirm ? null : { passwordMismatch: true };
}

const form = new FormGroup(
  {
    password: new FormControl(''),
    confirmPassword: new FormControl(''),
  },
  { validators: passwordMatchValidator },
);
```

## Dynamic Forms

Build forms dynamically based on data models:

```typescript
@Component({ ... })
export class DynamicFormComponent {
  private fb = inject(FormBuilder);
  fields = input<FieldConfig[]>();
  form = new FormGroup({});

  constructor() {
    effect(() => {
      const fields = this.fields();
      if (fields) {
        for (const field of fields) {
          this.form.addControl(field.name, new FormControl('', field.validators));
        }
      }
    });
  }
}
```
