# Protocols [![Travis](https://img.shields.io/travis/zkat/protocols.svg)](https://travis-ci.org/zkat/protocols) [![npm version](https://img.shields.io/npm/v/@zkat/protocols.svg)](https://npm.im/@zkat/protocols) [![license](https://img.shields.io/npm/l/@zkat/protocols.svg)](https://npm.im/@zkat/protocols)

[`protocols`](https://github.com/zkat/protocols) is a JavaScript library for
defining and implementing typeclass-like protocols that support dispatch across
multiple arguments. See also: [Clojure
protocols](http://clojure.org/reference/protocols) but on any number of types
across all arguments of the individual functions.

`protocols` is built on top of [`genfun`](https://npm.im/genfun), a fast,
prototype-based multimethod library, but this is mostly all behind the scenes.

On top of providing a nice, clear interface for defining interfaces, this
module makes the best effort to verify both protocols and implementations and
tries to provide clear, useful errors for common mistakes.

## Install

`$ npm install @zkat/protocols`

## Table of Contents

* [Example](#example)
* [API](#api)
  * [`protocol()`](#protocol)
  * [`implementation`](#impl)

### Example

```javascript
import protocol from '@zkat/protocols'

const Eq = protocol(['a', 'b'], {
  equal: ['a', 'b'],
  notEqual: ['a', 'b', (a, b) => !Eq.equal(a, b)]
})

class Foo {}
class Bar {}

Eq([Foo, Bar], {
  equal(a, b) { return a.name === b.name }
})

const foo = new Foo()
const bar = new Bar()
foo.name = bar.name = 'Alex'

Eq.equal(foo, bar) // true
Eq.notEqual(foo, bar) // false
Eq.equal(foo, foo) // Error: No protocol impl for `equal`
                   // found for arguments of types: (Foo, Foo)
```

### API

#### <a name="protocol"></a> `protocol(<types>, <spec>)`

Defines a new protocol across `<types>`, which will expect implementations for
the functions specified in `<spec>`.

The types in `<spec>` must map, by string name, to the type names specified in
`<types>`. The types in `<spec>` will then be used to map between method
implementations for the individual functions, and the provided types in the
impl.

##### Example

```javascript
const Eq = protocol(['a', 'b'], {
  eq: ['a', 'b']
})
```

#### <a name="impl"></a> `proto(<types>, <implementations>)`

Adds a new implementation to the given `proto` across `<types>`.

`<implementations>` must be an object with functions matching the protocol's
API. The types in `<types>` will be used for defining specific methods using
the function as the body.

If a protocol is derivable -- that is, all its functions have default impls,
then the `<implementations>` object can be omitted entirely, and the protocol
will be automatically derived for the given `<types>`

##### Example

```javascript
const Eq = protocol(['a', 'b'], {
  eq: ['a', 'b', (a, b) => a === b],
  neq: ['a', 'b', (a, b) => a !== b],
})

Eq([Number, Number], {
  eq(x, y) {
    return x === y
  }
})

Eq([String, String]) // Eq is derivable
```
