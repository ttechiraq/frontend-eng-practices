# Naming

## Identifiers

Identifiers must use only ASCII letters, digits, underscores (for constants and structured test method names), and (rarely) the '$' sign.

## Naming style

TypeScript expresses information in types, so names should not be decorated with information that is included in the type. (See also [Testing Blog](https://testing.googleblog.com/2017/10/code-health-identifiernamingpostforworl.html) for more about what not to include.)

Some concrete examples of this rule:

- Do not use trailing or leading underscores for private properties or methods.
- Do not use the `opt_` prefix for optional parameters.
  - For accessors, see [accessor rules](#getters-and-setters-accessors) below.
- Do not mark interfaces specially (`IMyInterface` or `MyFooInterface`) unless it's idiomatic in its environment. When introducing an interface for a class, give it a name that expresses why the interface exists in the first place (e.g. class `TodoItem` and `interface TodoItemStorage` if the interface expresses the format used for storage/serialization in JSON).
- Suffixing `Observable`s with `$` is a common external convention and can help resolve confusion regarding observable values vs concrete values. Judgement on whether this is a useful convention is left up to individual teams, but should be consistent within projects.

## Descriptive names

Names must be descriptive and clear to a new reader. Do not use abbreviations that are ambiguous or unfamiliar to readers outside your project, and do not abbreviate by deleting letters within a word.

- Exception: Variables that are in scope for 10 lines or fewer, including arguments that are not part of an exported API, may use short (e.g. single letter) variable names.

```typescript
// Good identifiers:
errorCount // No abbreviation.
dnsConnectionIndex // Most people know what "DNS" stands for.
referrerUrl // Ditto for "URL".
customerId // "Id" is both ubiquitous and unlikely to be misunderstood.

// Disallowed identifiers:
n // Meaningless.
nErr // Ambiguous abbreviation.
nCompConns // Ambiguous abbreviation.
wgcConnections // Only your group knows what this stands for.
pcReader // Lots of things can be abbreviated "pc".
cstmrId // Deletes internal letters.
kSecondsPerDay // Do not use Hungarian notation.
customerID // Incorrect camelcase of "ID".
```

## Camel case

Treat abbreviations like acronyms in names as whole words, i.e. use `loadHttpUrl`, not `loadHTTPURL`, unless required by a platform name (e.g. `XMLHttpRequest`).

## Dollar sign

Identifiers should not generally use `$`, except when required by naming conventions for third party frameworks. [See above](#naming-style) for more on using `$` with `Observable` values.

## Rules by identifier type

Most identifier names should follow the casing in the table below, based on the identifier's type.

| Style | Category |
|-------|----------|
| `UpperCamelCase` | class / interface / type / enum / decorator / type parameters / component functions in TSX / JSXElement type parameter |
| `lowerCamelCase` | variable / parameter / function / method / property / module alias |
| `CONSTANT_CASE` | global constant values, including enum values. See [Constants](#constants) |
| `#ident` | private identifiers are never used. |

## Type parameters

Type parameters, like in `Array<T>`, may use a single upper case character (`T`) or `UpperCamelCase`.

## Test names

Test method names in xUnit-style test frameworks may be structured with `_` separators, e.g. `testX_whenY_doesZ()`.

## `_` prefix/suffix

Identifiers must not use `_` as a prefix or suffix.

This also means that `_` must not be used as an identifier by itself (e.g. to indicate a parameter is unused).

Tip: If you only need some of the elements from an array (or TypeScript tuple), you can insert extra commas in a destructuring statement to ignore in-between elements:

```typescript
const [a, , b] = [1, 5, 10]; // a <- 1, b <- 10
```

## Imports

Module namespace imports are `lowerCamelCase` while files are `snake_case`, which means that imports correctly will not match in casing style, such as

```typescript
import * as fooBar from './foo_bar';
```

Some libraries might commonly use a namespace import prefix that violates this naming scheme, but overbearingly common open source use makes the violating style more readable. The only libraries that currently fall under this exception are:

## Constants

**Immutable:** `CONSTANT_CASE` indicates that a value is intended to not be changed, and may be used for values that can technically be modified (i.e. values that are not deeply frozen) to indicate to users that they must not be modified.

```typescript
const UNIT_SUFFIXES = {
  'milliseconds': 'ms',
  'seconds': 's',
};
// Even though per the rules of JavaScript UNIT_SUFFIXES is
// mutable, the uppercase shows users to not modify it.
```

A constant can also be a `static readonly` property of a class.

```typescript
class Foo {
  private static readonly MY_SPECIAL_NUMBER = 5;
  bar() {
    return 2 * Foo.MY_SPECIAL_NUMBER;
  }
}
```

**Global:** Only symbols declared on the module level, static fields of module level classes, and values of module level enums, may use `CONST_CASE`. If a value can be instantiated more than once over the lifetime of the program (e.g. a local variable declared within a function, or a static field on a class nested in a function) then it must use `lowerCamelCase`.

If a value is an arrow function that implements an interface, then it may be declared `lowerCamelCase`.

## Aliases

When creating a local-scope alias of an existing symbol, use the format of the existing identifier. The local alias must match the existing naming and format of the source. For variables use `const` for your local aliases, and for class fields use the `readonly` attribute.

Note: If you're creating an alias just to expose it to a template in your framework of choice, remember to also apply the proper [access modifiers].

```typescript
const {BrewStateEnum} = SomeType;
const CAPACITY = 5;

class Teapot {
  readonly BrewStateEnum = BrewStateEnum;
  readonly CAPACITY = CAPACITY;
}
```