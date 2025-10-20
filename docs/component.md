---
sidebar_position: 2
---


# Angular Components

In Angular, **components** are the building blocks of your application UI.  
They control *what* appears on the screen and *how* it behaves.

---

## 🧩 What is a Component?

A component in Angular is a **TypeScript class** decorated with `@Component()`.  
It defines:

- A **selector** — the custom HTML tag name.
- A **template** — the HTML structure.
- A **style** — the CSS applied to that component.


*in angular cli you can use `ng generate component {componentName}` to generate component with all file needed in 1 command.


```ts title="example: parent.component.ts"
import { Component } from '@angular/core';

@Component({
  selector: 'app-parent',
  templateUrl: './parent.component.html',
  styleUrl: './parent.component.scss'
})
export class ParentComponent {

}
```
---
## 🎯 Communicating Between Components

Often, components need to share data.
For example, a parent component might pass data down to a child, or a child might notify the parent about an event.

Angular provides two decorators for this:

**@Input()** — Receive data from parent to child

**@Output()** — Send data from child to parent

example

``` parent.component.html
<app-child [parentMsg]="sliderValue" (clicked)="onChildClicked($event)"></app-child>
```

``` child.component.ts
export class ChildComponent {
    @Input() parentMsg: any;
    @Output() clicked = new EventEmitter<string>();
}
```