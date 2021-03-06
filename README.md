# Protocols [![Travis](https://img.shields.io/travis/zkat/protocols.svg)](https://travis-ci.org/zkat/protocols) [![npm version](https://img.shields.io/npm/v/@zkat/protocols.svg)](https://npm.im/@zkat/protocols) [![license](https://img.shields.io/npm/l/@zkat/protocols.svg)](https://npm.im/@zkat/protocols)

[`@zkat/protocols`](https://github.com/zkat/protocols) is a JavaScript library
is a library for making groups of methods, called "protocols", that work
together to provide some abstract functionality that other things can then rely
on. If you're familiar with the concept of ["duck
typing"](https://en.wikipedia.org/wiki/Duck_typing), then it might make sense to
think of protocols as things that explicitly define what methods you need in
order to "clearly be a duck".

On top of providing a nice, clear interface for defining these protocols, this
module clear, useful errors when implementations are missing something or doing
something wrong.

One thing that sets this library apart from others is that on top of defining
duck-typed protocols on a single class/type, it lets you have different
implementations depending on the _arguments_. So a method on `Foo` may call
different code dependent on whether its first _argument_ is `Bar` or `Baz`. If
you've ever wished a method worked differently for different types of things
passed to it, this does that!

## Install

`$ npm install @zkat/protocols`

## Table of Contents

* [Example](#example)
* [API](#api)
  * [`protocol()`](#protocol)
  * [`implementation`](#impl)

### Example

```javascript
import protocol from "@zkat/protocols"

// Quackable is a protocol that defines three methods
const Quackable = protocol({
  walk: [],
  talk: [],
  isADuck: []
})

// `duck` must implement `Quackable` for this function to work. It doesn't
// matter what type or class duck is, as long as it implements Quackable.
function doStuffToDucks (duck) {
  if (!duck.isADuck()) {
    throw new Error('I want a duck!')
  } else {
    console.log(duck.walk())
    console.log(duck.talk())
  }
}

// elsewhere in the project...
class Person () {}

Quackable(Person, {
  walk() { return "my knees go the wrong way but I'm doing my best" }
  talk() { return "uhhh... do I have to? oh... 'Quack' 😒"}
  isADuck() { return true /* lol I'm totally lying */ }
})

// and another place...
class Duck () {}

Quackable(Duck, {
  walk() { return "*hobble hobble*" }
  talk() { return "QUACK QUACK" }
  isADuck() { return true }
})

// main.js
doStuffToDucks(new Person()) // works
doStuffToDucks(new Duck()) // works
doStuffToDucks({ walk() { return 'meh' } }) // => error
```

### API

#### <a name="protocol"></a> `protocol(<types>?, <spec>)`

Defines a new protocol on across arguments of types defined by `<types>`, which
will expect implementations for the functions specified in `<spec>`.

If `<types>` is missing, it will be treated the same as if it were an empty
array.


The types in `<spec>` must map, by string name, to the type names specified in
`<types>`, or be an empty array if `<types>` is omitted. The types in `<spec>`
will then be used to map between method implementations for the individual
functions, and the provided types in the impl.

##### Example

```javascript
const Eq = protocol(['a', 'b'], {
  eq: ['a', 'b']
})
```

#### <a name="impl"></a> `proto(<target>?, <types>?, <implementations>)`

Adds a new implementation to the given `proto` across `<types>`.

`<implementations>` must be an object with functions matching the protocol's
API. The types in `<types>` will be used for defining specific methods using
the function as the body.

Protocol implementations must include either `<target>`, `<types>`, or both:

* If only `<target>` is present, implementations will be defined the same as
  "traditional" methods -- that is, the definitions in `<implementations>`
  will add function properties directly to `<target>`.

* If only `<types>` is present, the protocol will keep all protocol functions as
  "static" methods on the protocol itself.

* If both are specified, protocol implementations will add methods to the `<target>`, and define multimethods using `<types>`.

If a protocol is derivable -- that is, all its functions have default impls,
then the `<implementations>` object can be omitted entirely, and the protocol
will be automatically derived for the given `<types>`

##### Example

```javascript
import protocol from '@zkat/protocols'

// Singly-dispatched protocols
const Show = protocol({
  show: []
})

class Foo {}

Show(Foo, {
  show () { return `[object Foo(${this.name})]` }
})

var f = new Foo()
f.name = 'alex'
f.show() === '[object Foo(alex)]'
```

```javascript
import protocol from '@zkat/protocols'

// Multi-dispatched protocols
const Comparable = protocol(['target'], {
  compare: ['target'],
})

class Foo {}
class Bar {}
class Baz {}

Comparable(Foo, [Bar], {
  compare (bar) { return 'bars are ok' }
})

Comparable(Foo, [Baz], {
  compare (baz) { return 'but bazzes are better' }
})

const foo = new Foo()
const bar = new Bar()
const baz = new Baz()

foo.compare(bar) // 'bars are ok'
foo.compare(baz) // 'but bazzes are better'
```
