---
sidebar_position: 8
---

# Comments and documentation

## JSDoc versus comments

There are two types of comments, JSDoc (`/** ... */`) and non-JSDoc ordinary comments (`// ...` or `/* ... */`).

- Use `/** JSDoc */` comments for documentation, i.e. comments a user of the code should read.
- Use `// line comments` for implementation comments, i.e. comments that only concern the implementation of the code itself.

JSDoc comments are understood by tools (such as editors and documentation generators), while ordinary comments are only for other humans.

## Multi-line comments

Multi-line comments are indented at the same level as the surrounding code. They must use multiple single-line comments (`//`-style), not block comment style (`/* */`).

```typescript
// This is
// fine

/*
* This should
* use multiple
* single-line comments
*/

/* This should use // */
```

Comments are not enclosed in boxes drawn with asterisks or other characters.

## JSDoc general form

The basic formatting of JSDoc comments is as seen in this example:

```typescript
/**
 * Multiple lines of JSDoc text are written here,
 * wrapped normally.
 * @param arg A number to do something to.
 */
function doSomething(arg: number) { … }
```

or in this single-line example:

```typescript
/** This short jsdoc describes the function. */
function doSomething(arg: number) { … }
```

If a single-line comment overflows into multiple lines, it must use the multi-line style with `/**` and `*/` on their own lines.

Many tools extract metadata from JSDoc comments to perform code validation and optimization. As such, these comments must be well-formed.

## Markdown

JSDoc is written in Markdown, though it may include HTML when necessary.

This means that tooling parsing JSDoc will ignore plain text formatting, so if you did this:

```typescript
/**
 * Computes weight based on three factors:
 * items sent
 * items received
 * last timestamp
 */
```

it will be rendered like this:

Computes weight based on three factors: items sent items received last timestamp

Instead, write a Markdown list:

```typescript
/**
 * Computes weight based on three factors:
 *
 * - items sent
 * - items received
 * - last timestamp
 */
```

## JSDoc tags

Google style allows a subset of JSDoc tags. Most tags must occupy their own line, with the tag at the beginning of the line.

```typescript
/**
 * The "param" tag must occupy its own line and may not be combined.
 * @param left A description of the left param.
 * @param right A description of the right param.
 */
function add(left: number, right: number) { ... }
```

```typescript
/**
 * The "param" tag must occupy its own line and may not be combined.
 * @param left @param right
 */
function add(left: number, right: number) { ... }
```

## Line wrapping

Line-wrapped block tags are indented four spaces. Wrapped description text may be lined up with the description on previous lines, but this horizontal alignment is discouraged.

```typescript
/**
 * Illustrates line wrapping for long param/return descriptions.
 * @param foo This is a param with a particularly long description that just
 *     doesn't fit on one line.
 * @return This returns something that has a lengthy description too long to fit
 *     in one line.
 */
exports.method = function(foo) {
  return 5;
};
```

Do not indent when wrapping a `@desc` or `@fileoverview` description.

## Document all top-level exports of modules

Use `/** JSDoc */` comments to communicate information to the users of your code. Avoid merely restating the property or parameter name. You should also document all properties and methods (exported/public or not) whose purpose is not immediately obvious from their name, as judged by your reviewer.

Exception: Symbols that are only exported to be consumed by tooling, such as @NgModule classes, do not require comments.

## Class comments

JSDoc comments for classes should provide the reader with enough information to know how and when to use the class, as well as any additional considerations necessary to correctly use the class. Textual descriptions may be omitted on the constructor.

## Method and function comments

Method, parameter, and return descriptions may be omitted if they are obvious from the rest of the method's JSDoc or from the method name and type signature.

Method descriptions begin with a verb phrase that describes what the method does. This phrase is not an imperative sentence, but instead is written in the third person, as if there is an implied `This method ...` before it.

## Parameter property comments

A [parameter property](https://www.typescriptlang.org/docs/handbook/2/classes.html#parameter-properties) is a constructor parameter that is prefixed by one of the modifiers `private`, `protected`, `public`, or `readonly`. A parameter property declares both a parameter and an instance property, and implicitly assigns into it. For example, `constructor(private readonly foo: Foo)`, declares that the constructor takes a parameter `foo`, but also declares a private readonly property `foo`, and assigns the parameter into that property before executing the remainder of the constructor.

To document these fields, use JSDoc's `@param` annotation. Editors display the description on constructor calls and property accesses.

```typescript
/** This class demonstrates how parameter properties are documented. */
class ParamProps {
  /**
   * @param percolator The percolator used for brewing.
   * @param beans The beans to brew.
   */
  constructor(
      private readonly percolator: Percolator,
      private readonly beans: CoffeeBean[]) {}
}

/** This class demonstrates how ordinary fields are documented. */
class OrdinaryClass {
  /** The bean that will be used in the next call to brew(). */
  nextBean: CoffeeBean;

  constructor(initialBean: CoffeeBean) {
    this.nextBean = initialBean;
  }
}
```

## JSDoc type annotations

JSDoc type annotations are redundant in TypeScript source code. Do not declare types in `@param` or `@return` blocks, do not write `@implements`, `@enum`, `@private`, `@override` etc. on code that uses the `implements`, `enum`, `private`, `override` etc. keywords.

## Make comments that actually add information

For non-exported symbols, sometimes the name and type of the function or parameter is enough. Code will usually benefit from more documentation than just variable names though!

Avoid comments that just restate the parameter name and type, e.g.

```typescript
/** @param fooBarService The Bar service for the Foo application. */
```

Because of this rule, `@param` and `@return` lines are only required when they add information, and may otherwise be omitted.

```typescript
/**
 * POSTs the request to start coffee brewing.
 * @param amountLitres The amount to brew. Must fit the pot size!
 */
brew(amountLitres: number, logger: Logger) {
  // ...
}
```

## Comments when calling a function

"Parameter name" comments should be used whenever the method name and parameter value do not sufficiently convey the meaning of the parameter.

Before adding these comments, consider refactoring the method to instead accept an interface and destructure it to greatly improve call-site readability.

Parameter name comments go before the parameter value, and include the parameter name and a `=` suffix:

```typescript
someFunction(obviousParam, /* shouldRender= */ true, /* name= */ 'hello');
```

Existing code may use a legacy parameter name comment style, which places these comments ~after~ the parameter value and omits the `=`. Continuing to use this style within the file for consistency is acceptable.

```typescript
someFunction(obviousParam, true /* shouldRender */, 'hello' /* name */);
```

## Place documentation prior to decorators

When a class, method, or property have both decorators like `@Component` and JsDoc, please make sure to write the JsDoc before the decorator.

Do not write JsDoc between the Decorator and the decorated statement.

```typescript
@Component({ selector: 'foo', template: 'bar', })
/** Component that prints "bar". */
export class FooComponent {}
```

Write the JsDoc block before the Decorator.

```typescript
/** Component that prints "bar". */
@Component({ selector: 'foo', template: 'bar', })
export class FooComponent {}
```