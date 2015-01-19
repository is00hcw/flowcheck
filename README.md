This library has 2 main modules:

- a **source transformer**
- a **runtime assertion library**

The transformer adds to your code an assert for each (Flow) type.

The runtime assertion module checks the types at runtime. If an assert fails **the debugger kicks in** so you can inspect the stack and quickly find out what's wrong.

In the runtime assertion module, a type is represented by an object with 2 properties:

- name: string
- validate: function

The validate function has the following signature:

```js
validate(x: any, ctx: ?Array<any>, fast: ?boolean): ?Array<Failure>
```

where

```js
Failure = {
  actual: any;
  expected: Type;
  path: Array<string | number>;
}
```

So you can also use flowcheck as a **general purpose validation** library.

# Transformations

## Basic types

- number
- string
- boolean
- void
- any
- mixed

```js
var f = require('flowcheck/assert');

var x: type = y;
// =>
var x = f.check(y, f.<type>);
```

## Arrays

```js
var x: Array<T> = y;
// or
var x: T[] = y;
// =>
var x = f.check(y, f.list(T));
```

## Tuples

```js
var x: [T1, T2, ...] = y;
// =>
var x = f.check(y, f.tuple([T1, T2, ...]));
```

## Classes

Classes start with a uppercase letter.

```js
var x: T = y;
// =>
var x = f.check(y, T);
```

## Objects

```js
var x: {p1: T1; p2: T2; ... pn: Tn;} = y;
// =>
var x = f.check(y, f.object({p1: T1, p2: T2, ... pn: Tn}));
```

## Dictionaries

```js
var x: {[key:D]: C} = y;
// =>
var x = f.check(y, f.dict(D, C));
```

## Maybe Types

```js
var x: ?T = y;
// =>
var x = f.check(y, f.maybe(T));
```

## Unions

```js
var x: T1 | T2 | ... | Tn = y;
// =>
var x = f.check(y, f.union([T1, T2, ... , Tn]));
```

## Functions

```js
function f(x1: T1, x2: T2, ... , xn: Tn): R {
  return x;
}
// =>
function f(x1: T1, x2: T2, ... , xn: Tn): R {
  f.check(arguments, f.args([T1, T2, ... , Tn]));
  var ret = (function (x1, x2, ... , xn) {
    return x;
  }).apply(this, arguments);
  return f.check(ret, R);
}
```

# Type aliases

```js
type T = Array<string>;
// =>
var T = f.list(f.string);
```

# API

`transform(source: string, options: ?object): string`

Inserts the type assertions.

Options

- "typeAssertionModule": string (default `flowcheck`)
- "typeAssertionVariable": string (default `f`)

# Assert library API

`f.irreducible(name: string, is: (x: any) => boolean): Type`

Defines a new irreducible type,

`f.check(value: T, type: Type): T`

- by default `instanceof` is used to check the type
- if `type` owns a static `is(x: any)`, it will be used  to check the type
- if `value` is not of type `type`, `f.fail` will be called with a meaningful error message

`f.list(type: Type, name: ?string): Type`

Returns a type representing the list `Array<type>`.

`f.tuple(types: Array<Type>, name: ?string): Type`

Returns a type representing the tuple `[...types]`.

`f.object(props: {[key:string]: Type}): Type`

Returns a type representing the object `{p1: T1, p2: T2, ... p3: T3}`.

`f.dict(domain: Type, codomain: Type, name: ?string): Type`

Returns a type representing the nullable type `?type`.

`f.maybe(type: Type, name: ?string): Type`

Returns a type representing the nullable type `?type`.

`f.union(types: Array<Type>, name: ?string): Type`

Returns a type representing the union `T1 | T2 | ... | Tn`.
