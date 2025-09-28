---
sidebar_position: 7
---

# Toolchain requirements

Google style requires using a number of tools in specific ways, outlined here.

## TypeScript compiler

All TypeScript files must pass type checking using the standard tool chain.

## @ts-ignore

Do not use `@ts-ignore` nor the variants `@ts-expect-error` or `@ts-nocheck`.

**Why?**

They superficially seem to be an easy way to fix a compiler error, but in practice, a specific compiler error is often caused by a larger problem that can be fixed more directly.

For example, if you are using `@ts-ignore` to suppress a type error, then it's hard to predict what types the surrounding code will end up seeing. For many type errors, the advice in [how to best use any](#any) is useful.

You may use `@ts-expect-error` in unit tests, though you generally should not. `@ts-expect-error` suppresses all errors. It's easy to accidentally over-match and suppress more serious errors. Consider one of:

- When testing APIs that need to deal with unchecked values at runtime, add casts to the expected type or to `any` and add an explanatory comment. This limits error suppression to a single expression.
- Suppress the lint warning and document why, similar to [suppressing `any` lint warnings](#any-suppress).

## Conformance

Google TypeScript includes several conformance frameworks, [tsetse](https://tsetse.info) and [tsec](https://github.com/google/tsec).

These rules are commonly used to enforce critical restrictions (such as defining globals, which could break the codebase) and security patterns (such as using `eval` or assigning to `innerHTML`), or more loosely to improve code quality.

Google-style TypeScript must abide by any applicable global or framework-local conformance rules.