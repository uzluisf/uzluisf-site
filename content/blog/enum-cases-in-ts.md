---
title: "Enum Cases Are Its Own Types in TypeScript"
date: "2023-04-15T19:46:01-04:00"
draft: false
slug: 'enum-cases-own-types-in-ts'
tags: ['enums', 'typescript']
---

The other day I was adding a feature flag to a page within a React codebase. The
relevant piece of code looked as follows:

```typescript
enum Light {
  string Green = 'Green',
  string Yellow = 'Yellow',
  string Red = 'Red',
};

declare function getStatus(): Light;
const status = getStatus();

switch (status) {
  case Light.Green:
    console.log('render view 1');
  case Light.Yellow:
    console.log('render view 2 (with feature flag)');
  case Light.Red:
    console.log('render view 3');
}
```


Since I wanted to hit only one of the switch cases, specifically `Light.Yellow`,
my first instinct was to set `status` to `Light.Yellow` and then I could test
the feature flag on that page:

```typescript
const status = Light.Yellow;
```

However doing this caused the following compilation error:

```typescript
Type 'Light.Green' is not comparable to type 'Light.Yellow'. (2678)
Type 'Light.Red' is not comparable to type 'Light.Yellow'.(2678)
```

In TypeScript, each enumeration case is its own type and when I assigned
`Light.Yellow` to `status`, TS had enough information to discern that `status`
wouldn't be anything else other than `Light.Yellow`. However using type
assertion with `as`, I could override TypeScript's type inference and get the
behavior I wanted:

```typescript
const status = Light.Yellow as Light;
```

Now `status`'s type was `Light` instead of `Light.Yellow`, and it could match
any of the enum cases and hence no compilation error.
