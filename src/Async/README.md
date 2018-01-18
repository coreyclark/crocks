# Async

```haskell
Async e a = Rejected e | Resolved a
```

Defined as a "lazy" Sum Type that implements an asynchronous control structure,
`Async` represents either the success or failure of a given asynchronous flow.
The "laziness" of `Async` allows the ability to build complex asynchronous
operations by defining each portion of that flow as smaller "steps" that are
composed together using the all to familiar monadic interface.

Depending on your needs, an `Async` can be constructed in a variety of ways. The
typical closely resembles how a typical `Promise` is constructed with one major
difference, the arguments used in the function that is passed to the `Promise`
constructor are reversed in an `Async`.

There are many way to represent asynchronous operations in javascript, and as
such, many libraries available to us in our ecosystem provide different means
to take advantage of these operations. The two most common use
either `Promise` returning functions or allow for the Continuation Passing Style
prevalent in the asynchronous function that ship with node. `Async` provides to
construction helpers that can wrap functions that use these styles of
asynchronous processing and will give you back a function that takes the same
arguments as the original and will return an `Async` instead. These functions
are called [`fromPromise`](#frompromise) and [`fromNode`](#fromnode).

[ Forking ]

[ Cancellation ]


```javascript
```

## Implements
`Functor`, `Alt`,  `Bifunctor`, `Apply`, `Chain`, `Applicative`, `Monad`

## Constructor Methods

#### Rejected

```haskell
Async.Rejected :: e -> Async e a
```

Used to construct a `Rejected` instance of `Async` that represents the failure
or "false" portion of the disjunction. Calling `Rejected` with a given
value, will return a new `Rejected` instance, wrapping the provided value.

When an instance is `Rejected`, most `Async` returning methods on the instance
will return another `Rejected` instance. This is in contrast to a
javascript `Promise`, that will continue on a `Resolved` path after
a `catch`. This behavior of `Promise`s make provide challenges when constructing
complicated (or even some simple) `Promise` chains that may fail at various
steps along the chain.

Even though `Async` is a `Bifunctor`, in most cases it is desired to keep the
type of a `Rejected` fixed to a type for a given flow. Given that `Async` is
a `Bifunctor`, it is easy to make sure you get the type you need at the edge
by leaning on [`bimap`](#bimap) to square things up.

```javascript
```

#### Resolved

```haskell
Async.Resolved :: a -> Async e a
```

Used to construct a `Resolved` instance that represents the success or "true"
portion of the disjunction. `Resolved` will wrap any given value passed to this
constructor in the `Resolved` instance it returns, signaling the validity of the
wrapped value.

```javascript
```

#### fromPromise

```haskell
Async.fromPromise :: (* -> Promise a e) -> (* -> Async e a)
```

Used to turn an "eager" `Promise` returning function, into a function that takes
the same arguments but returns a "lazy" `Async` instance instead.

```javascript
```

#### fromNode

```haskell
NodeCallback :: (e, a) -> ()
Async.fromNode :: ((*, NodeCallback) -> ()) -> (* -> Async e a)
Async.fromNode :: (((*, NodeCallback) -> ()), ctx) -> (* -> Async e a)
```

[ desc ]

```javascript
```

#### all

```haskell
Async.all :: [ Async e a ] -> Async e [ a ]
```

[ desc ]

```javascript
```

#### of

```haskell
Async.of :: a -> Async e a
```

Used to wrap any value into an `Async` as a `Resolved` instance, `of` is
used mostly by helper functions that work "generically" with instances of
either `Applicative` or `Monad`. When working specifically with
the `Async` type, the [`Resolved`](#resolved) constructor should be
used. Reach for `of` when working with functions that will work with
ANY `Applicative`/`Monad`.

```javascript
```

## Instance Methods

#### map

```haskell
Async e a ~> (a -> b) -> Async e b
```

Used to apply transformations to `Resolved` values of an `Async`, `map` takes
a function that it will lift into the context of the `Async` and apply to it
the wrapped value. When ran on a `Resolved` instance, `map` will apply the
wrapped value to the provided function and return the result in a
new `Resolved` instance.


```javascript
```

#### alt

```haskell
Async e a ~> Async e a -> Async e a
```

Providing a means for a fallback or alternative value, `alt` combines (2)
`Async` instances and will return the first `Resolved` instance it encounters
or a `Rejected` instance if it does not encounter a `Resolved`.

```javascript
```

#### bimap

```haskell
Async e a ~> ((e -> b), (a -> c)) -> Async b c
```

[ desc ]

```javascript
```

#### ap

```haskell
Async e (a -> b) ~> Async e a -> Async e b
```

Short for apply, `ap` is used to apply an `Async` instance containing a value
to another `Async` instance that contains a function, resulting in
new `Async` instance with the result. `ap` requires that it is called on
an instance that is either `Rejected` or `Resolved` that wraps a curried
polyadic function.

When either `Async` is `Rejected`, `ap` will return a `Rejected` instance, that
wraps the value of the original `Rejected` instance. This can be used to safely
combine multiple values under a given combination function. If any of the inputs
result in a `Rejected` than they will never be applied to the function and will
not result in undesired exceptions or results.

When [`fork`](#fork)ed, all `Async`s chained with multiple `ap` invocations
will be executed concurrently.

```javascript
```

#### chain

```haskell
Async e a ~> (a -> Async e b) -> Async e b
```

Combining a sequential series of transformations that capture disjunction can be
accomplished with `chain`. `chain` expects a unary, `Async` returning function
as its argument. When invoked on a `Rejected` instance , `chain` will not run
the function, but will instead return another `Rejected` instance with the
original `Rejected` value. When called on a `Resolved` instance however, the
inner value will be passed to provided function, returning the result as the
new instance.

```javascript
```

#### coalesce

```haskell
Async e a ~> ((e -> b), (a -> b))) -> Async e b
```
Used as a means to apply a computation to a `Resolved` instance and then map
any `Rejected` value and transform it to a `Resolved` to continue
computation. `coalesce` on an `Async` can be used to model the all too
familiar, and more imperative `if/else` flow in a more declarative manner.

The first function is used when invoked on a `Rejected` instance and will
return a `Resolved` instance wrapping the result of the function. The second
function is used when `coalesce` is invoked on a `Resolved` instance and is used
to map the original value, returning a new `Resolved` instance wrapping the
result of the second function.

```javascript
```

#### swap

```haskell
Async e a ~> ((e -> b), (a -> c)) -> Async c b
```

[ desc ]

```javascript
```

#### fork

```haskell
Async e a ~> ((e -> ()), (a -> ())) -> (() -> ())
Async e a ~> ((e -> ()), (a -> ()), (() -> ())) -> (() -> ())
```

[ desc ]

```javascript
```

#### toPromise

```haskell
Async e a ~> () -> Promise a e
```

[ desc ]

```javascript
```

## Transformation Functions

#### eitherToAsync

`crocks/Async/eitherToAsync`

```haskell
eitherToAsync :: Either b a -> Async b a
eitherToAsync :: (a -> Either c b) -> a -> Async c b
```

Used to transform a given `Either` instance to
an `Async` instance, `eitherToAsync` will turn a `Right` instance into
a `Resolved` instance wrapping the original value contained in the
original `Right`. If a `Left` is provided, then `eitherToAsync` will return
a `Rejected` instance, wrapping the original `Left` value.

Like all `crocks` transformation functions, `eitherToAsync` has (2) possible
signatures and will behave differently when passed either an `Either` instance
or a function that returns an instance of `Either`. When passed the instance,
a transformed `Async` is returned. When passed an `Either` returning function,
a function will be returned that takes a given value and returns an `Async`.

```javascript
```

#### firstToAsync

`crocks/Async/firstToAsync`

```haskell
firstToAsync :: e -> First a -> Async e a
firstToAsync :: e -> (a -> First b) -> a -> Async e b
```

Used to transform a given `First` instance to
an `Async` instance, `firstToAsync` will turn a non-empty `First` instance into
a `Resolved` instance wrapping the original value contained in the
original non-empty.

The `First` datatype is based on a `Maybe` and as such its left or empty value
is fixed to a `()` type. As a means to allow for convenient
transformation, `firstToAsync` takes a default `Rejected` value as the first
argument. This value will be wrapped in a resulting `Rejected` instance in the
case of empty.

Like all `crocks` transformation functions, `firstToAsync` has (2) possible
signatures and will behave differently when passed either a `First` instance
or a function that returns an instance of `First`. When passed the instance,
a transformed `Async` is returned. When passed a `First` returning function,
a function will be returned that takes a given value and returns an `Async`.

```javascript
```

#### lastToAsync

`crocks/Async/lastToAsync`

```haskell
lastToAsync :: e -> Last a -> Async e a
lastToAsync :: e -> (a -> Last b) -> a -> Async e b
```

Used to transform a given `Last` instance to
an `Async` instance, `lastToAsync` will turn a non-empty `Last` instance into
a `Resolved` instance wrapping the original value contained in the
original non-empty.

The `Last` datatype is based on a `Maybe` and as such its left or empty value
is fixed to a `()` type. As a means to allow for convenient
transformation, `lastToAsync` takes a default `Rejected` value as the first
argument. This value will be wrapped in a resulting `Rejected` instance, in the
case of empty.

Like all `crocks` transformation functions, `firstToAsync` has (2) possible
signatures and will behave differently when passed either a `Last` instance
or a function that returns an instance of `Last`. When passed the instance,
a transformed `Async` is returned. When passed a `Last` returning function,
a function will be returned that takes a given value and returns an `Async`.

```javascript
```

#### maybeToAsync

`crocks/Async/maybeToAsync`

```haskell
maybeToAsync :: e -> Maybe a -> Async e a
maybeToAsync :: e -> (a -> Maybe b) -> a -> Async e b
```

Used to transform a given `Maybe` instance to
an `Async` instance, `maybeToAsync` will turn a `Just` instance into
a `Resolved` instance wrapping the original value contained in the
original `Just`.

A `Nothing` instance is fixed to a `()` type and as such can only ever contain
a value of `undefined`. As a means to allow for convenient
transformation, `maybeToAsync` takes a default `Rejected` value as the first
argument. This value will be wrapped in a resulting `Rejected` instance, in the
case of `Nothing`.

Like all `crocks` transformation functions, `maybeToAsync` has (2) possible
signatures and will behave differently when passed either a `Maybe` instance
or a function that returns an instance of `Maybe`. When passed the instance,
a transformed `Async` is returned. When passed a `Maybe` returning function,
a function will be returned that takes a given value and returns an `Async`.

```javascript
```

#### resultToAsync

`crocks/Async/resultToAsync`

```haskell
resultToAsync :: Result b a -> Async b a
resultToAsync :: (a -> Result c b) -> a -> Result c b
```

Used to transform a given `Result` instance to
an `Async` instance, `resultToAsync` will turn an `Ok` instance into
a `Resolved` instance wrapping the original value contained in the
original `Ok`. If an `Err` is provided, then `resultToAsync` will return
a `Rejected` instance, wrapping the original `Err` value.

Like all `crocks` transformation functions, `resultToAsync` has (2) possible
signatures and will behave differently when passed either a `Result` instance
or a function that returns an instance of `Result`. When passed the instance,
a transformed `Async` is returned. When passed a `Result` returning function,
a function will be returned that takes a given value and returns an `Async`.

```javascript
```