---
sidebar_position: 4
---

# Language features

This section delineates which features may or may not be used, and any additional constraints on their use.
Language features which are not discussed in this style guide may be used with no recommendations of their usage.

##

##

##

##

##

##

##

##

## hello

### hello1

### hello2

### hello3

### hello4

### Equality checks

Always use triple equals (`===`) and not equals (`!==`). The double equality operators cause error prone type coercions that are hard to understand and slower to implement for JavaScript Virtual Machines. See also the JavaScript equality table.

```ts
if (foo == "bar" || baz != bam) {
  // Hard to understand behaviour due to type coercion.
}
if (foo === "bar" || baz !== bam) {
  // All good here.
}
```

Exception: Comparisons to the literal null value may use the == and != operators to cover both null and undefined values.

```ts
if (foo == null) {
  // Will trigger when foo is null or undefined.
}
```

### Type and non-nullability assertions

Type assertions (`x as SomeType`) and non-nullability assertions (`y!`) are unsafe. Both only silence the TypeScript compiler, but do not insert any runtime checks to match these assertions, so they can cause your program to crash at runtime.

Because of this, you should not use type and non-nullability assertions without an obvious or explicit reason for doing so.

Instead of the following:

```ts
(x as Foo).foo();

y!.bar();
```

When you want to assert a type or non-nullability the best answer is to explicitly write a runtime check that performs that check.

```ts
// assuming Foo is a class.
if (x instanceof Foo) {
  x.foo();
}

if (y) {
  y.bar();
}
```

Sometimes due to some local property of your code you can be sure that the assertion form is safe. In those situations, you should add clarification to explain why you are ok with the unsafe behavior:

```ts
// x is a Foo, because ...
(x as Foo).foo();

// y cannot be null, because ...
y!.bar();
```

If the reasoning behind a type or non-nullability assertion is obvious, the comments may not be necessary. For example, generated proto code is always nullable, but perhaps it is well-known in the context of the code that certain fields are always provided by the backend. Use your judgement.

**Type assertion syntax**
Type assertions must use the `as` syntax (as opposed to the angle brackets syntax). This enforces parentheses around the assertion when accessing a member.

```ts
const x = (<Foo>z).length;
const y = <Foo>z.length;
```

```ts
// z must be Foo because ...
const x = (z as Foo).length;
```

**Double assertions**
From the TypeScript handbook, TypeScript only allows type assertions which convert to a more specific or less specific version of a type. Adding a type assertion (`x as Foo`) which does not meet this criteria will give the error: Conversion of type 'X' to type 'Y' may be a mistake because neither type sufficiently overlaps with the other.

If you are sure an assertion is safe, you can perform a double assertion. This involves casting through `unknown` since it is less specific than all types.

```ts
// x is a Foo here, because...
(x as unknown as Foo).fooMethod();
```

Use `unknown` (instead of `any` or `{}`) as the intermediate type.

Type assertions and object literals
Use type annotations (`: Foo`) instead of type assertions (`as Foo`) to specify the type of an object literal. This allows detecting refactoring bugs when the fields of an interface change over time.

```ts
interface Foo {
  bar: number;
  baz?: string; // was "bam", but later renamed to "baz".
}

const foo = {
  bar: 123,
  bam: "abc", // no error!
} as Foo;

function func() {
  return {
    bar: 123,
    bam: "abc", // no error!
  } as Foo;
}
```

```ts
interface Foo {
  bar: number;
  baz?: string;
}

const foo: Foo = {
  bar: 123,
  bam: "abc", // complains about "bam" not being defined on Foo.
};

function func(): Foo {
  return {
    bar: 123,
    bam: "abc", // complains about "bam" not being defined on Foo.
  };
}
```

### Keep try blocks focused

Limit the amount of code inside a try block, if this can be done without hurting readability.

```ts
try {
  const result = methodThatMayThrow();
  use(result);
} catch (error: unknown) {
  // ...
}
```

```ts
let result;
try {
  result = methodThatMayThrow();
} catch (error: unknown) {
  // ...
}
use(result);
```

Moving the non-throwable lines out of the try/catch block helps the reader learn which method throws exceptions. Some inline calls that do not throw exceptions could stay inside because they might not be worth the extra complication of a temporary variable.

**Exception**: There may be performance issues if try blocks are inside a loop. Widening try blocks to cover a whole loop is ok.

## Decorators

Decorators are syntax with an `@` prefix, like `@MyDecorator`.

Do not define new decorators. Only use the decorators defined by frameworks:

- Angular (e.g. `@Component`, `@Injectable`, etc.)

Why?

We generally want to avoid decorators, because they were an experimental feature that have since diverged from the TC39 proposal and have known bugs that won't be fixed.

When using decorators, the decorator must immediately precede the symbol it decorates, with no empty lines between:

```ts
/** JSDoc comments go before decorators */
@Component({...})  // Note: no empty line after the decorator.
class MyComp {
  @Input() myField: string;  // Decorators on fields may be on the same line...

  @Input()
  myOtherField: string;  // ... or wrap.
}
```

## Disallowed features

### Wrapper objects for primitive types

TypeScript code must not instantiate the wrapper classes for the primitive types `String`, `Boolean`, and `Number`. Wrapper classes have surprising behavior, such as `new Boolean(false)` evaluating to `true`.

```ts
const s = new String("hello");
const b = new Boolean(false);
const n = new Number(5);
```

The wrappers may be called as functions for coercing (which is preferred over using + or concatenating the empty string) or creating symbols. See type coercion for more information.

### Automatic Semicolon Insertion

Do not rely on Automatic Semicolon Insertion (ASI). Explicitly end all statements using a semicolon. This prevents bugs due to incorrect semicolon insertions and ensures compatibility with tools with limited ASI support (e.g. clang-format).

### Const enums

Code must not use `const enum;` use plain `enum` instead.

Why?

TypeScript enums already cannot be mutated; `const enum` is a separate language feature related to optimization that makes the enum invisible to JavaScript users of the module.

### Debugger statements

Debugger statements must not be included in production code.

```ts
function debugMe() {
  debugger;
}
```

### `with`

Do not use the `with` keyword. It makes your code harder to understand and has been banned in strict mode since ES5.

### Dynamic code evaluation

Do not use `eval` or the `Function(...string)` constructor (except for code loaders). These features are potentially dangerous and simply do not work in environments using strict Content Security Policies.

### Non-standard features

Do not use non-standard ECMAScript or Web Platform features.

This includes:

- Old features that have been marked deprecated or removed entirely from ECMAScript / the Web Platform (see MDN)
- New ECMAScript features that are not yet standardized
- - Avoid using features that are in current TC39 working draft or currently in the proposal process
- - Use only ECMAScript features defined in the current ECMA-262 specification
- Proposed but not-yet-complete web standards:
- - WHATWG proposals that have not completed the proposal process.
- Non-standard language “extensions” (such as those provided by some external transpilers)

Projects targeting specific JavaScript runtimes, such as latest-Chrome-only, Chrome extensions, Node.JS, Electron, can obviously use those APIs. Use caution when considering an API surface that is proprietary and only implemented in some browsers; consider whether there is a common library that can abstract this API surface away for you.

### Modifying builtin objects

Never modify builtin types, either by adding methods to their constructors or to their prototypes. Avoid depending on libraries that do this.

Do not add symbols to the global object unless absolutely necessary (e.g. required by a third-party API).
