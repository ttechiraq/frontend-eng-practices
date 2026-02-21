---
sidebar_position: 3
---

# Signals

Angular Signals is a system that granularly tracks how and where your state is used throughout an application, allowing the framework to optimize rendering updates.

Source: [angular.dev/guide/signals](https://angular.dev/guide/signals)

## What Are Signals?

A signal is a wrapper around a value that notifies interested consumers when that value changes. Signals can contain any value, from primitives to complex data structures.

You read a signal's value by calling its getter function, which allows Angular to track where the signal is used.

Signals may be either **writable** or **read-only**.

## Writable Signals

Writable signals provide an API for updating their values directly. You create writable signals by calling the `signal` function with the signal's initial value:

```typescript
const count = signal(0);

// Signals are getter functions — calling them reads their value.
console.log('The count is: ' + count());
```

To change the value of a writable signal, either `.set()` it directly:

```typescript
count.set(3);
```

or use the `.update()` operation to compute a new value from the previous one:

```typescript
// Increment the count by 1.
count.update((value) => value + 1);
```

Writable signals have the type `WritableSignal`.

### Converting Writable Signals to Read-Only

`WritableSignal` provides an `asReadonly()` method that returns a read-only version of the signal. This is useful when you want to expose a signal's value to consumers without allowing them to modify it directly:

```typescript
@Injectable({ providedIn: 'root' })
export class CounterState {
  // Private writable state
  private readonly _count = signal(0);

  // Public read-only signal
  readonly count = this._count.asReadonly();

  increment() {
    this._count.update((v) => v + 1);
  }
}

@Component({ /* ... */ })
export class AwesomeCounter {
  state = inject(CounterState);
  count = this.state.count; // can read but not modify

  increment() {
    this.state.increment();
  }
}
```

:::warning[important]
Read-only signals do not have any built-in mechanism that would prevent deep-mutation of their value.
:::

## Computed Signals

Computed signals are read-only signals that derive their value from other signals. You define computed signals using the `computed` function and specifying a derivation:

```typescript
const count: WritableSignal<number> = signal(0);
const doubleCount: Signal<number> = computed(() => count() * 2);
```

The `doubleCount` signal depends on the `count` signal. Whenever `count` updates, Angular knows that `doubleCount` needs to update as well.

### Lazily Evaluated and Memoized

`doubleCount`'s derivation function does not run to calculate its value until the first time you read `doubleCount`. The calculated value is then cached, and if you read `doubleCount` again, it returns the cached value without recalculating.

If you then change `count`, Angular knows that `doubleCount`'s cached value is no longer valid, and the next time you read `doubleCount` its new value will be calculated.

As a result, you can safely perform computationally expensive derivations in computed signals, such as filtering arrays.

### Dynamic Dependencies

Only the signals actually read during the derivation are tracked. For example, in this computed the `count` signal is only read if the `showCount` signal is true:

```typescript
const showCount = signal(false);
const count = signal(0);
const conditionalCount = computed(() => {
  if (showCount()) {
    return `The count is ${count()}.`;
  } else {
    return 'Nothing to see here!';
  }
});
```

When you read `conditionalCount`, if `showCount` is `false`, the "Nothing to see here!" message is returned without reading the `count` signal. This means that updates to `count` will not result in a recomputation of `conditionalCount`.

## Reactive Contexts

A reactive context is a runtime state where Angular monitors signal reads to establish a dependency. The code reading the signal is the consumer, and the signal being read is the producer.

Angular automatically enters a reactive context when:

- Rendering a component template (including bindings in the host property)
- Evaluating a `resource`'s params or loader function
- Evaluating a `linkedSignal`
- Evaluating a `computed` signal
- Executing an `effect` or `afterRenderEffect` callback

During these operations, Angular creates a live connection. If a tracked signal changes, Angular will eventually re-run the consumer.

### Reading Without Tracking Dependencies

Rarely, you may want to read a signal without creating a dependency. You can do this with `untracked`:

```typescript
effect(() => {
  console.log(`User set to ${currentUser()} and the counter is ${untracked(counter)}`);
});
```

`untracked` is also useful when an effect needs to invoke external code that shouldn't be treated as a dependency:

```typescript
effect(() => {
  const user = currentUser();
  untracked(() => {
    // If the loggingService reads signals, they won't be counted
    // as dependencies of this effect.
    this.loggingService.log(`User set to ${user}`);
  });
});
```

### Reactive Context and Async Operations

The reactive context is only active for synchronous code. Any signal reads that occur after an asynchronous boundary will not be tracked as dependencies.

```typescript
// Avoid: theme() won't be tracked after await
effect(async () => {
  const data = await fetchUserData();
  console.log(`User: ${data.name}, Theme: ${theme()}`);
});

// Prefer: read signals before await
effect(async () => {
  const currentTheme = theme(); // Read before await
  const data = await fetchUserData();
  console.log(`User: ${data.name}, Theme: ${currentTheme}`);
});
```

## Dependent State with linkedSignal

`linkedSignal` creates a writable signal whose value is derived from other signals but can also be set manually. When its source signals change, the linked signal recomputes its value.

```typescript
const options = signal(['apple', 'banana', 'cherry']);
const selectedOption = linkedSignal(() => options()[0]);

console.log(selectedOption()); // 'apple'

// Can be set manually
selectedOption.set('banana');
console.log(selectedOption()); // 'banana'

// When source changes, linked signal recomputes
options.set(['date', 'elderberry', 'fig']);
console.log(selectedOption()); // 'date'
```

`linkedSignal` is useful for state that has a default derived from other state but can be overridden by user interaction.

Source: [angular.dev/guide/signals/linked-signal](https://angular.dev/guide/signals/linked-signal)

## Async Reactivity with Resources

All signal APIs are synchronous — `signal`, `computed`, `input`, etc. However, applications often need to deal with data that is available asynchronously. A `Resource` gives you a way to incorporate async data into your application's signal-based code.

```typescript
import { resource, signal } from '@angular/core';

const userId = signal(1);

const userResource = resource({
  params: () => ({ id: userId() }),
  loader: async ({ params }) => {
    const response = await fetch(`/api/users/${params.id}`);
    return response.json();
  },
});

// Access the resource's state
console.log(userResource.value());    // The loaded data
console.log(userResource.status());   // 'idle' | 'loading' | 'resolved' | 'error'
console.log(userResource.error());    // Error if status is 'error'
```

### httpResource

For HTTP requests specifically, Angular provides `httpResource` which integrates with `HttpClient`:

```typescript
import { httpResource } from '@angular/common/http';

const usersResource = httpResource<User[]>({
  url: '/api/users',
});
```

Source: [angular.dev/guide/signals/resource](https://angular.dev/guide/signals/resource)

## Effects

An `effect` runs a function whenever one or more signals it reads change. Effects are useful for side effects that interact with non-reactive APIs (logging, analytics, DOM manipulation).

```typescript
import { effect, signal } from '@angular/core';

const count = signal(0);

effect(() => {
  console.log(`The count is: ${count()}`);
});
```

Effects run at least once. When any of the signals read inside the effect change, the effect re-executes.

### afterRenderEffect

For side effects that need access to the DOM, use `afterRenderEffect`:

```typescript
import { afterRenderEffect, ElementRef, inject } from '@angular/core';

@Component({ ... })
export class ChartComponent {
  private el = inject(ElementRef);

  constructor() {
    afterRenderEffect(() => {
      // Safe to access and manipulate the DOM here
      updateChart(this.el.nativeElement);
    });
  }
}
```

Source: [angular.dev/guide/signals/effect](https://angular.dev/guide/signals/effect)

## Signal Equality Functions

When creating a signal, you can optionally provide an equality function, which is used to check whether the new value is actually different from the previous one:

```typescript
import _ from 'lodash';

const data = signal(['test'], { equal: _.isEqual });

// Even though this is a different array instance, the deep equality
// function considers the values to be equal, so the signal won't
// trigger any updates.
data.set(['test']);
```

By default, signals use referential equality (`Object.is()` comparison).

## Type Checking Signals

You can use `isSignal` to check if a value is a Signal, and `isWritableSignal` to check if it is writable:

```typescript
const count = signal(0);
const doubled = computed(() => count() * 2);

isSignal(count);          // true
isSignal(doubled);        // true
isSignal(42);             // false

isWritableSignal(count);   // true
isWritableSignal(doubled); // false
```

## Signals in OnPush Components

When you read a signal within an `OnPush` component's template, Angular tracks the signal as a dependency of that component. When the value of that signal changes, Angular automatically marks the component for check to ensure it gets updated the next time change detection runs.

## Using Signals with RxJS

Angular provides interoperability between signals and RxJS via `@angular/core/rxjs-interop`:

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

// Convert an Observable to a Signal
const count$ = interval(1000);
const count = toSignal(count$, { initialValue: 0 });

// Convert a Signal to an Observable
const name = signal('Angular');
const name$ = toObservable(name);
```

Source: [angular.dev/guide/ecosystem/rxjs-interop](https://angular.dev/guide/ecosystem/rxjs-interop)
