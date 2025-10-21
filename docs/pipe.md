---
sidebar_position: 3
---

# Angular Pipe

Help optimize the performance of Angular applications by transforming data in templates or HTML.

## What is a Pipe?

A Pipe in Angular is a way to transform data before displaying it in the view. Pipes are simple functions that take in data as input and return transformed data as output. They are used in Angular templates with the pipe operator (`|`).

## Built-in Pipes

Angular provides several built-in pipes for common data transformations:

- **DatePipe**: Formats dates according to locale rules.
- **UpperCasePipe**: Transforms text to uppercase.
- **LowerCasePipe**: Transforms text to lowercase.
- **CurrencyPipe**: Formats numbers as currency.
- **DecimalPipe**: Formats numbers with decimal points.

```typescript
// in template
{{ currentDate | date:'fullDate' }} // Formats currentDate to full date format
{{ amount | currency:'USD' }} // Formats amount as USD currency
{{ text | uppercase }} // Transforms text to uppercase
```

## Custom Pipes

You can also create your own custom pipes by implementing the `PipeTransform` interface.

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
    name: 'getSportDesc',
    pure: true // default is true, set to false for impure pipe
})
export class GetSportDescPipe implements PipeTransform {
    transform(sportId: number, sports: any[]): string {
        return sports.find(sport => sport.sportId === sportId)?.description || 'Unknown Sport';
    }
})

// Declare Pipe in component
@Component({
    selector: 'app-sport',
    template: `
        <div *ngFor="let sport of sports">
            {{ sport.sportId | getSportDesc:sports }}
        </div>
    `,
    providers: [GetSportDescPipe]
})
export class SportComponent { }
```

## Pure vs Impure Pipes

By default, all pipes are considered pure, which means that it only executes when a primitive input value (such as a String, Number, Boolean, or Symbol) or a object reference (such as Array, Object, Function, or Date) is changed. Pure pipes offer a performance advantage because Angular can avoid calling the transformation function if the passed value has not changed.

When you want a pipe to detect changes within arrays or objects, it must be marked as an impure function by passing the pure flag with a value of false.

```typescript
import { Pipe, PipeTransform } from "@angular/core"
@Pipe({
	name: "joinNamesImpure",
	pure: false,
})
export class JoinNamesImpurePipe implements PipeTransform {
	transform(names: string[]): string {
		return names.join()
	}
}
```
