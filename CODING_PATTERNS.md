<p align="center">
  <img src="assets/logo.png" alt="Appsweet Logo" width="250" height="auto" />
</p>

<h1 align="center">Coding Patterns</h1>

## General

### Delete Dead Code

Code comments are an [anti-pattern](https://kentcdodds.com/blog/please-dont-commit-commented-out-code). Always delete dead code. Use [git-diff](https://git-scm.com/docs/git-diff) to restore old code.

### Avoid the Wrong Abstraction

Duplicating code is better than abstracting code in the wrong way. See [this article](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) for why.

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

### Constant Objects

Use [Constant Objects](https://www.typescriptlang.org/docs/handbook/enums.html#objects-vs-enums) instead of Enums. Use [String Enums](https://www.typescriptlang.org/docs/handbook/enums.html#string-enums) if needed.

```ts
// Best ✔✔
export const CONFIG = {
  FOO: 'FOO',
  BAR: 'BAR'
} as const;

// Good ✔
export enum Config {
  Foo = 'Foo',
  Bar = 'Bar'
}

// Avoid ✘
export enum Config {
  Foo,
  Bar
}

// Avoid ✘
export const enum Config {
  Foo,
  Bar
}

// Avoid ✘
export const enum Config {
  Foo = 'Foo',
  Bar = 'Bar'
}
```


## CSS

Coming soon

## HTML

Coming soon
