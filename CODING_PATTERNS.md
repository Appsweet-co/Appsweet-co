<p align="center">
  <img src="assets/logo.png" alt="Appsweet Logo" width="250" height="auto" />
</p>

<h1 align="center">Coding Patterns</h1>

## General

### Delete Dead Code

Code comments are an [anti-pattern](https://kentcdodds.com/blog/please-dont-commit-commented-out-code). Always delete dead code. Use [git-diff](https://git-scm.com/docs/git-diff) to restore old code.

### Avoid the Wrong Abstraction

Duplicating code is better than abstracting code in the wrong way. See [this article](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) for why.

### Avoid Sacred Code

Blindly duplicating code leads to [The Wrong Abstraction](#avoid-the-wrong-abstraction). Always improve the code to meet your needs.

### Avoid Code Comments

Code comments are an [anti-pattern](https://kentcdodds.com/blog/please-dont-commit-commented-out-code). Use [Descriptive and Meaningful Phrases](#descriptive-and-meaningful-phrases) instead.

## TypeScript

### Type Driven Development

Consider [writing your types first](https://www.olioapps.com/blog/type-driven-development-with-typescript/). This makes it easy to catch bugs at compile time and helps Intelisense work better during development. 

### Type Aliases and Interfaces

Generic [type annotations](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) like `string` and `number` can be hard to work with. They prevent type safety and stop Intellisense from working correctly. Use [type aliases](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases), [interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#interfaces), and [nominal types](#nominal-types) when possible.

```ts
// Good ✔
export interface CommonListItem {
  id: UUIDString;
  timestamp: ISOString;
}

// Good ✔
private getNotificationSettings(response: NotificationResponse) {
  // ...
}

// Avoid ✘
export interface CommonListItem {
  id: string;
  timestamp: string;
}

// Avoid ✘
private getNotificationSettings(response: any) {
  // ...
}
```

### Nominal Types

Nominal Types add symantic meaning to what would otherwise be a generic type. This makes it easy to tell the difference between things like timestamps and UUIDs when reading and writing code.

Nominal Types come in two flavors: **Flexible** and **Strict**.

Intellisense treats **Flexible Nominal Types** as unique, but TypeScript treats them as equivalent to their underlying generic type. This prevents compile time errors and preserves backward compatabilty with existing code. Use a Flexible Nominal Type when in doubt.

```ts
type Foo = Nominal<'Foo', string>;
type Bar = string;

Foo === Bar
// => true
```

Both Intellisense and TypeScript treat **Strict Nominal Types** as unique. They are never equivalent to their underlying generic type. This breaks backward compatabilty with existing code, but guarantees an extra level of type safety. Use Strict Nominal Types with caution.

```ts
type Foo = NominalStrict<'Foo', string>;
type Bar = string;

Foo === Bar
// => false
```

:dart: ***PRO TIP: See these articles to learn more about [Flexible Nominal Types](https://spin.atomicobject.com/2018/01/15/typescript-flexible-nominal-typing/) and [Strict Nominal Types](https://www.typescriptlang.org/play#example/nominal-typing).***

### Const Assertions

Use [const assertions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions) instead of Enums. Use [String Enums](https://www.typescriptlang.org/docs/handbook/enums.html#string-enums) if needed.

```ts
// Best ✔✔
const CONFIG = {
  FOO: 'FOO',
  BAR: 'BAR'
} as const;

// Good ✔
enum Config {
  Foo = 'Foo',
  Bar = 'Bar'
}

// Avoid ✘
enum Config {
  Foo,
  Bar
}

// Avoid ✘
const enum Config {
  Foo,
  Bar
}

// Avoid ✘
const enum Config {
  Foo = 'Foo',
  Bar = 'Bar'
}
```

### Immutable Data

Mutated data is hard to work with. Use [immutable data](https://en.wikipedia.org/wiki/Immutable_object) instead.

```ts
// Best ✔✔
const listA = [ 1, 2, 3, 0, 4, 5 ];
const listB = [ ...listA, 6, 7, 8 ];

// Good ✔
const listA = [ 1, 2, 3, 0, 4, 5 ];
const listB = listA.concat([ 6, 7, 8 ]);

// Avoid ✘
const listA = [ 1, 2, 3, 0, 4, 5 ];
listA.push(6, 7, 8);
```

### Pure Functions

[Pure Functions](https://en.wikipedia.org/wiki/Pure_function) are easy to test and work well with immutable data. They're also [referentially transparent](https://www.yld.io/blog/the-not-so-scary-guide-to-functional-programming/) and easy for JavaScript engines to [optimize](https://v8.dev/blog/turbofan-jit).

```ts
// Good ✔
const add = (x, y) => y + x;

// Good ✔
const add = (x) => (y) => y + x;

// Avoid ✘
const add = (x) => x + magicNumberFromSomewhere;

// Avoid ✘
const add = () => magicNumberFromSomewhere + magicNumberFromSomewhereElse;
```

### Parameter Defaults

Set defaults for any function arguments with a single common value.

```ts
// Good ✔
const api = (endpoint, path = "foo/bar/baz") => `${path}/${endpoint}`;

// Avoid ✘
const api = (endpoint, path?) => path ? `${path}/${endpoint}` : `foo/bar/baz/${endpoint}`;
```

### Graceful Fallbacks

Return fallbacks instead of throwing errors or exceptions.

```ts
// Good ✔
const data = await getData(url).catch(() => []);

// Avoid ✘
const data = await getData(url).catch(msg => console.error(msg));
```

:dart: ***PRO TIP: Use [Type Assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions) to set the correct return type for fallbacks.***

### Fallback First, Data Last

Passing data as the [last argument](https://dev.to/richytong/practical-functional-programming-in-javascript-data-last-1gjo) of a function is great for piping and currying, but TypeScript's typing system works best when we pass in data as the [first argument](https://basarat.gitbook.io/typescript/type-system/type-inference).

Consider passing a [fallback value](#graceful-fallbacks) as the first argument and actual data as the last.

```ts
// Good ✔
const applyOr = (fallback, fn, data) => fn(data) ?? fallback;

// Good ✔
const applyOr = (fallback) => (fn) => (data) => fn(data) ?? fallback;

// Avoid ✘
const applyOr = (data, fallback, fn) => fn(data) ?? fallback;

// Avoid ✘
const applyOr = (fn) => (data) => (fallback) => fn(data) ?? fallback;
```

### Function Expressions

Functions should always [return a value](#pure-functions). Function statements don't return values and make code hard to work with. Use function statements strategically.

```ts
// Good ✔
const data = await getData(url).catch(() => []);
const formatted = format(data);

// Avoid ✘
const data = await getData(url).catch(() => []);
formatData();
```

### Conditional (Ternary) Operators

Use [Conditional (Ternary) Operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) instead of nested `if...else` statements.

```ts
// Good ✔
const getAuthenticatedPath = (endpoint) => {
  return (endpoint === 'dashboard') ? `/user/auth/${endpoint}` : `/user/auth/account/${endpoint}`;
}

const getUnauthenticatedPath = (endpoint) => {
  return (endpoint === 'dashboard') ? `/guest/${endpoint}` : `/guest/account/${endpoint}`;
}

const getFullPath = (endpoint, isAuthenticated) => {
  return (isAuthenticated) ? getAuthenticatedPath(endpoint) : getUnauthenticatedPath(endpoint);
}

// Avoid ✘
const getFullPath = (endpoint, isAuthenticated) => {
  if (isAuthenticated) {
    if (endpoint === 'dashboard') {
      return `/user/auth/${endpoint}`;
    } 
    
    if (endpoint !== 'dashboard') {
        return `/user/auth/account/${endpoint}`;
    }
  }
    
  return `/guest/${endpoint}`;
}
```

### Strict Equality

Use [Strict Equality](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness#strict_equality_using) to compare values. [Loose Equality](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness#loose_equality_using) can lead to unwanted side effects.

```ts
// Good ✔
const result = (foo === bar) ? baz : baff;

// Avoid ✘
const result = (foo == bar) ? baz : baff;
```

### Short-Circuit Evaluation

Use [Short-Circuit Evaluation](https://www.educative.io/answers/what-are-javascript-short-circuiting-operators) to avoid executing unnecessary code.

```ts
// Good ✔
const validated = hasUppercase(password) && hasNumber(password);

// Avoid ✘
const validated = [hasUppercase(password), hasNumber(password)].every(isTrue);
```

### Descriptive And Meaningful Phrases

Abbreviations are hard to work with. Make your code easy to read. Use [Descriptive And Meaningful Phrases](https://medium.com/mutual-of-omaha-digital-experience-and-design-team/damp-programming-reviving-readability-d84647cc5b2e) when possible.

```ts
// Good ✔
const getNextPage = (current) => (current === 'suggested') ? 'homepage' : 'suggested';

// Avoid ✘
const next = (c) => (c === 'sp') ? 'hp' : 'sp';
```

### Private Members, Local Variables

Use [private class members](https://www.typescriptlang.org/docs/handbook/2/classes.html#private) and [local variables](https://developer.mozilla.org/en-US/docs/Glossary/Local_variable) when possible. This makes it easy to [delete dead code](#delete-dead-code).

### Use Doc Blocks

Consider adding [JSDocs](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html) for your public APIs in addition to [type annotations](#type-driven-development). 

Doc blocks are a good [developer experience](https://medium.com/swlh/what-is-dx-developer-experience-401a0e44a9d9). They help devs understand what an API does and why they might use it.

Also consider adding [`@example`](https://jsdoc.app/tags-example.html) use cases in your doc blocks.

```ts
/**
 * Returns the sum of two numbers. Same as `y + x`.
 *
 * @example
 *
 * ```ts
 * add(3)(6);
 * // => 9
 * ```
 */
export const add = (x: number) => (y: number) => y + x;
```

## CSS

### Native CSS

Use native CSS when possible. Native [functions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Functions) and [variables](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) are easier to override than variables stored in preprocessors like Sass.

```scss
/* Good ✔ */
:root {
  --scale-sm: 0.5rem;
  --scale-md: 1rem;
  --scale-lg: 2rem;
}

/* Avoid ✘ */
$scale-sm: 0.5rem;
$scale-md: 1rem;
$scale-lg: 2rem;
```

## HTML

Coming soon
