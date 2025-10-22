---
sidebar_position: 6
---

# Angular Forms

have two types of form in Angular (one more in the future with Signal based form)

- Template-driven form
- Reactive form

## Template-driven Form

Basic, easy to use, less code in component class, suitable for simple form with less validation logic.
Use **ngModel** to create two-way data bindings for reading and writing input-control values.

Here's an example of a template-driven form with email validation:

```typescript
// email-form.component.ts
import { Component } from "@angular/core"
import { NgForm } from "@angular/forms"

@Component({
	selector: "app-email-form",
	templateUrl: "./email-form.component.html",
})
export class EmailFormComponent {
	userEmail = "" // This will hold our email value

	onSubmit(form: NgForm) {
		if (form.valid) {
			console.log("Email submitted:", this.userEmail)
		}
	}
}
```

```html
<!-- email-form.component.html -->
<form #emailForm="ngForm" (ngSubmit)="onSubmit(emailForm)">
	<div>
		<label for="email">Email:</label>
		<input
			type="email"
			id="email"
			name="email"
			[(ngModel)]="userEmail"
			#email="ngModel"
			required
			email
		/>

		<!-- Show validation messages -->
		<div *ngIf="email.invalid && (email.dirty || email.touched)">
			<div *ngIf="email.errors?.['required']">Email is required</div>
			<div *ngIf="email.errors?.['email']">
				Please enter a valid email address
			</div>
		</div>
	</div>

	<button type="submit" [disabled]="emailForm.invalid">Submit</button>
</form>
```

Remember to import `FormsModule` in your module:

```typescript
import { NgModule } from "@angular/core"
import { FormsModule } from "@angular/forms"

@NgModule({
	imports: [
		FormsModule, // Add this to enable template-driven forms
	],
	// ...
})
export class AppModule {}
```

This example demonstrates:

- Two-way data binding with `[(ngModel)]`
- Form validation with `required` and `email` validators
- Error messages using template variables
- Form submission handling
- Disable submit button when form is invalid

## Reactive Form

More powerful and scalable, suitable for complex forms with dynamic validation logic.
Uses `FormControl`, `FormGroup`, and `FormBuilder` to create form controls in the component class.

Here's an example of a Reactive Form for user information:

```typescript
// user.interface.ts
export interface User {
	userId?: number
	firstName: string
	lastName: string
}

// user-form.component.ts
import { Component, OnInit } from "@angular/core"
import { FormBuilder, FormGroup, Validators } from "@angular/forms"
import { User } from "./user.interface"

@Component({
	selector: "app-user-form",
	templateUrl: "./user-form.component.html",
})
export class UserFormComponent implements OnInit {
	userForm: FormGroup

	constructor(private fb: FormBuilder) {}

	ngOnInit() {
		this.userForm = this.fb.group({
			userId: [""], // optional field
			firstName: ["", [Validators.required, Validators.minLength(2)]],
			lastName: ["", [Validators.required, Validators.minLength(2)]],
		})
	}

	// Getter methods for easy access in template
	get firstName() {
		return this.userForm.get("firstName")
	}
	get lastName() {
		return this.userForm.get("lastName")
	}

	onSubmit() {
		if (this.userForm.valid) {
			const user: User = this.userForm.value
			console.log("User submitted:", user)
		}
	}

	// Optional: Load existing user data
	loadUser(user: User) {
		this.userForm.patchValue(user)
	}
}
```

```html
<!-- user-form.component.html -->
<form [formGroup]="userForm" (ngSubmit)="onSubmit()">
	<div>
		<label for="firstName">First Name:</label>
		<input id="firstName" type="text" formControlName="firstName" />
		<div *ngIf="firstName?.invalid && (firstName?.dirty || firstName?.touched)">
			<div *ngIf="firstName?.errors?.['required']">First name is required</div>
			<div *ngIf="firstName?.errors?.['minlength']">
				First name must be at least 2 characters
			</div>
		</div>
	</div>

	<div>
		<label for="lastName">Last Name:</label>
		<input id="lastName" type="text" formControlName="lastName" />
		<div *ngIf="lastName?.invalid && (lastName?.dirty || lastName?.touched)">
			<div *ngIf="lastName?.errors?.['required']">Last name is required</div>
			<div *ngIf="lastName?.errors?.['minlength']">
				Last name must be at least 2 characters
			</div>
		</div>
	</div>

	<button type="submit" [disabled]="userForm.invalid">Submit</button>
</form>
```

Remember to import `ReactiveFormsModule` in your module:

```typescript
import { NgModule } from "@angular/core"
import { ReactiveFormsModule } from "@angular/forms"

@NgModule({
	imports: [
		ReactiveFormsModule, // Add this to enable reactive forms
	],
	// ...
})
export class AppModule {}
```

Key features demonstrated:

- Form initialization using `FormBuilder`
- Form validation with built-in validators
- Getter methods for cleaner template code
- Error handling and display
- Type-safe form submission with interface
- Optional data loading with `patchValue`

## Nested Forms

Here's an example of composing forms using separate components with interfaces:

```typescript
// interfaces/address.interface.ts
export interface Address {
	street: string
	city: string
	zipCode: string
}

// interfaces/user.interface.ts
// Base user information
export interface User {
	userId?: number
	firstName: string
	lastName: string
}

// interfaces/user-address.interface.ts
import { User } from "./user.interface"
import { Address } from "./address.interface"

export type UserWithAddress = User & {
	address: Address
}

// components/address-form/address-form.component.ts
import { Component, OnInit } from "@angular/core"
import {
	FormBuilder,
	FormGroup,
	Validators,
	FormGroupDirective,
} from "@angular/forms"

@Component({
	selector: "app-address-form",
	templateUrl: "./address-form.component.html",
})
export class AddressFormComponent implements OnInit {
	// Define address form group
    addressForm: FormGroup

	constructor(
		private fb: FormBuilder,
		private parentForm: FormGroupDirective
	) {}

	ngOnInit() {
		// Option 1: Bind to existing parent FormGroup
		const parentGroup = this.parentForm.form.get("address") as FormGroup
		this.addressForm = parentGroup
		// Ensure controls exist on the parent group (in case parent initialized empty group)
		if (!this.addressForm.get("street")) {
			this.addressForm.addControl(
				"street",
				this.fb.control("", Validators.required)
			)
		}
		if (!this.addressForm.get("city")) {
			this.addressForm.addControl(
				"city",
				this.fb.control("", Validators.required)
			)
		}
		if (!this.addressForm.get("zipCode")) {
			this.addressForm.addControl(
				"zipCode",
				this.fb.control("", [
					Validators.required,
					Validators.pattern("^[0-9]{5}$"),
				])
			)
		}

		//// Option 2: Or initialize a new address FormGroup then add to new formGroup to parent
		// this.addressForm = this.fb.group({
		// street: ['', Validators.required],
		// city: ['', Validators.required],
		// zipCode: ['', [Validators.required, Validators.pattern('^[0-9]{5}$')]]
		// });
		//// Add address form to parent form
		// this.parentForm.form.addControl('address', this.addressForm);
	}

	// Getter methods for template access
	get street() {
		return this.addressForm.get("street")
	}
	get city() {
		return this.addressForm.get("city")
	}
	get zipCode() {
		return this.addressForm.get("zipCode")
	}
}
```

```html
<!-- components/address-form/address-form.component.html -->
<div [formGroup]="addressForm">
	<div>
		<label for="street">Street:</label>
		<input id="street" type="text" formControlName="street" />
		<div *ngIf="street?.invalid && (street?.dirty || street?.touched)">
			<div *ngIf="street?.errors?.['required']">Street is required</div>
		</div>
	</div>

	<div>
		<label for="city">City:</label>
		<input id="city" type="text" formControlName="city" />
		<div *ngIf="city?.invalid && (city?.dirty || city?.touched)">
			<div *ngIf="city?.errors?.['required']">City is required</div>
		</div>
	</div>

	<div>
		<label for="zipCode">ZIP Code:</label>
		<input id="zipCode" type="text" formControlName="zipCode" />
		<div *ngIf="zipCode?.invalid && (zipCode?.dirty || zipCode?.touched)">
			<div *ngIf="zipCode?.errors?.['required']">ZIP code is required</div>
			<div *ngIf="zipCode?.errors?.['pattern']">ZIP code must be 5 digits</div>
		</div>
	</div>
</div>
```

```typescript
// components/user-profile-form/user-profile-form.component.ts
import { Component, OnInit } from "@angular/core"
import { FormBuilder, FormGroup, Validators } from "@angular/forms"
import { User } from "../../interfaces/user.interface"
import {
	UserWithAddress,
	UserAddress,
	UserProfile,
} from "../../interfaces/user-address.interface"

@Component({
	selector: "app-user-profile-form",
	templateUrl: "./user-profile-form.component.html",
})
export class UserProfileFormComponent implements OnInit {
	userProfileForm: FormGroup

	constructor(private fb: FormBuilder) {}

	ngOnInit() {
		this.userProfileForm = this.fb.group({
			userId: [""],
			firstName: ["", [Validators.required, Validators.minLength(2)]],
			lastName: ["", [Validators.required, Validators.minLength(2)]],
			address: this.fb.group({}), // Address form group will be created in child component
		})
	}

	get firstName() {
		return this.userProfileForm.get("firstName")
	}
	get lastName() {
		return this.userProfileForm.get("lastName")
	}

	onSubmit() {
		if (this.userProfileForm.valid) {
			// Option 1: Using UserWithAddress type
			const userWithAddress: UserWithAddress = this.userProfileForm.value
			console.log("User Profile submitted (UserWithAddress):", userWithAddress)

			// You can also transform between formats if needed
			const transformToUserProfile: UserProfile = {
				user: {
					userId: userWithAddress.userId,
					firstName: userWithAddress.firstName,
					lastName: userWithAddress.lastName,
				},
				address: userWithAddress.address,
			}
		}
	}
}
```

```html
<!-- components/user-profile-form/user-profile-form.component.html -->
<form [formGroup]="userProfileForm" (ngSubmit)="onSubmit()">
	<div>
		<label for="firstName">First Name:</label>
		<input id="firstName" type="text" formControlName="firstName" />
		<div *ngIf="firstName?.invalid && (firstName?.dirty || firstName?.touched)">
			<div *ngIf="firstName?.errors?.['required']">First name is required</div>
			<div *ngIf="firstName?.errors?.['minlength']">
				First name must be at least 2 characters
			</div>
		</div>
	</div>

	<div>
		<label for="lastName">Last Name:</label>
		<input id="lastName" type="text" formControlName="lastName" />
		<div *ngIf="lastName?.invalid && (lastName?.dirty || lastName?.touched)">
			<div *ngIf="lastName?.errors?.['required']">Last name is required</div>
			<div *ngIf="lastName?.errors?.['minlength']">
				Last name must be at least 2 characters
			</div>
		</div>
	</div>

	<!-- Address Form Component -->
	<div formGroupName="address">
		<h3>Address</h3>
		<app-address-form></app-address-form>
	</div>

	<button type="submit" [disabled]="userProfileForm.invalid">Submit</button>
</form>
```

Key features of this implementation:

- Separate interfaces for User and Address
- Reusable AddressForm component
- Proper form group nesting
- Validation at both parent and child levels
- Type safety with interfaces
- Clean separation of concerns

Note on parent binding approaches:

- Constructor injection (used in the `AddressFormComponent` above):

  - We inject `FormGroupDirective` in the child's constructor: `constructor(private fb: FormBuilder, private parentForm: FormGroupDirective)`.
  - The child component then reads `this.parentForm.form.get('address')` and binds to that FormGroup. This is explicit and works well when you want to directly manipulate or ensure controls exist on the parent group.
  - Pros: Explicit, easy to reason about, works well with DI and unit tests.
  - Cons: Child depends on being used inside a parent form; you'll get runtime errors if used standalone (default in new version).

- viewProviders / ControlContainer approach (earlier example):
  - Uses `viewProviders` to provide the `ControlContainer` to the child so the child template can use relative form directives without manually grabbing the parent form instance.
  - Pros: Cleaner templates and decouples the child from parent implementation details.
  - Cons: Slightly more magical; harder to unit-test the child in complete isolation without wiring a host form.

Choose the approach that fits your team's preference and test strategy. For strict explicitness and easier testability, constructor-injected `FormGroupDirective` is a solid choice. For cleaner templates and looser coupling, `viewProviders` + `ControlContainer` is convenient.

---

Flat-composition examples (UserWithAddress)

Below are two separate examples that implement the flat `UserWithAddress` type. Both create a parent form with an `address` FormGroup and show two ways the child `AddressFormComponent` can bind to that group.

1. viewProviders / ControlContainer (clean template, child uses relative directives)

```typescript
// components/address-form-via-controlcontainer/address-form-via-controlcontainer.component.ts
import { Component } from "@angular/core"
import { FormBuilder, FormGroup, Validators } from "@angular/forms"
import { ControlContainer, FormGroupDirective } from "@angular/forms"

@Component({
	selector: "app-address-form-cc",
	templateUrl: "./address-form-cc.component.html",
	viewProviders: [
		{ provide: ControlContainer, useExisting: FormGroupDirective },
	],
})
export class AddressFormViaControlContainerComponent {
	// no constructor injection of parent form necessary
	constructor(private fb: FormBuilder) {}
}
```

```html
<!-- components/address-form-via-controlcontainer/address-form-cc.component.html -->
<!-- Child uses relative formControlName bindings because ControlContainer is provided -->
<div>
	<label>Street</label>
	<input formControlName="street" />
	<label>City</label>
	<input formControlName="city" />
	<label>ZIP</label>
	<input formControlName="zipCode" />
</div>
```

Parent usage:

```typescript
// parent component that uses flat UserWithAddress
this.userProfileForm = this.fb.group({
	userId: [""],
	firstName: ["", Validators.required],
	lastName: ["", Validators.required],
	address: this.fb.group({
		street: ["", Validators.required],
		city: ["", Validators.required],
		zipCode: ["", [Validators.required, Validators.pattern("^[0-9]{5}$")]],
	}),
})
```

```html
<!-- parent template -->
<form [formGroup]="userProfileForm">
	<!-- user fields -->
	<div formGroupName="address">
		<app-address-form-cc></app-address-form-cc>
	</div>
</form>
```

2. Constructor-injected FormGroupDirective (explicit access)

```typescript
// components/address-form-via-parent/address-form-via-parent.component.ts
import { Component, OnInit } from "@angular/core"
import {
	FormBuilder,
	FormGroup,
	Validators,
	FormGroupDirective,
} from "@angular/forms"

@Component({
	selector: "app-address-form-parent",
	templateUrl: "./address-form-parent.component.html",
})
export class AddressFormViaParentComponent implements OnInit {
	addressForm: FormGroup

	constructor(
		private fb: FormBuilder,
		private parentForm: FormGroupDirective
	) {}

	ngOnInit() {
		const parentGroup = this.parentForm.form.get("address") as FormGroup
		if (parentGroup) {
			this.addressForm = parentGroup
		} else {
			this.addressForm = this.fb.group({
				street: ["", Validators.required],
				city: ["", Validators.required],
				zipCode: ["", [Validators.required, Validators.pattern("^[0-9]{5}$")]],
			})
		}
	}
}
```

```html
<!-- components/address-form-via-parent/address-form-parent.component.html -->
<div [formGroup]="addressForm">
	<label>Street</label>
	<input formControlName="street" />
	<label>City</label>
	<input formControlName="city" />
	<label>ZIP</label>
	<input formControlName="zipCode" />
</div>
```

Parent usage is identical to the viewProviders example: place the child inside a parent `formGroupName="address"` container. The difference is the child explicitly reads the parent form instance in code.

Which to choose

- Use viewProviders/ControlContainer when you prefer cleaner child templates and looser coupling.
- Use FormGroupDirective injection when you want explicit control and easier programmatic manipulation of the parent group (adding controls dynamically, setting validators from the child, etc.).
