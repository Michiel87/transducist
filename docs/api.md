# API

## Table of Contents

<!-- toc -->

- [Starting a chain](#starting-a-chain)
  * [`chainFrom(iterable)`](#chainfromiterable)
  * [`transducerBuilder()`](#transducerbuilder)
- [Transformation methods](#transformation-methods)
  * [`.dedupe()`](#dedupe)
  * [`.drop(n)`](#dropn)
  * [`.dropWhile(pred)`](#dropwhilepred)
  * [`.filter(pred)`](#filterpred)
  * [`.flatMap(f)`](#flatmapf)
  * [`.interpose(separator)`](#interposeseparator)
  * [`.keep(f)`](#keepf)
  * [`.map(f)`](#mapf)
  * [`.partitionAll(n)`](#partitionalln)
  * [`.partitionBy(f)`](#partitionbyf)
  * [`.remove(pred)`](#removepred)
  * [`.take(n)`](#taken)
  * [`.takeNth(n)`](#takenthn)
  * [`.takeWhile(pred)`](#takewhilepred)
  * [`.compose(transducer)`](#composetransducer)
- [Ending a chain](#ending-a-chain)
  * [`.count()`](#count)
  * [`.every(pred)`](#everypred)
  * [`.find(pred)`](#findpred)
  * [`.first()`](#first)
  * [`.forEach(f)`](#foreachf)
  * [`.isEmpty()`](#isempty)
  * [`.some(pred)`](#somepred)
  * [`.stringJoin(separator)`](#stringjoinseparator)
  * [`.toArray()`](#toarray)
  * [`.toIterator()`](#toiterator)
  * [`.reduce(reducer, intialValue?)`](#reducereducer-intialvalue)
- [Reducers](#reducers)
  * [`toSum()`](#tosum)
  * [`toAverage()`](#toaverage)
  * [`toMin(comparator?)`](#tomincomparator)
  * [`toMax(comparator?)`](#tomaxcomparator)
  * [`toObject()`](#toobject)
- [Utility functions](#utility-functions)
  * [`makeTransducer(f: (reducer, result, input) => result)`](#maketransducerf-reducer-result-input--result)
  * [`reduced(result)`](#reducedresult)

<!-- tocstop -->

## Starting a chain

### `chainFrom(iterable)`

Starts a chain. Any number of transformation methods may be added, after which a
termination method should be called to produce a result. No computation is done
until a termination method is called.

The argument may be any iterable, including an array or a string. The argument
may also be an object, in which case it is treated as an iterable of key-value
pairs, each a two element array.

### `transducerBuilder()`

Starts a chain for constructing a new transducer. Any number of trasformation
methods may be added, after which `.build()` should be called to produce a
transducer.

## Transformation methods

Any number of these methods may be called on a chain to add transformations in
sequence.

### `.dedupe()`

Removes elements that are equal to the proceeding element (using `===` for
equality). For example:
```ts
chainFrom([1, 2, 2, 3, 3, 3])
    .dedupe()
    .toArray(); // -> [1, 2, 3]
```

### `.drop(n)`

Skips the first `n` elements. For example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .drop(3)
    .toArray(); // -> [4, 5]
```

### `.dropWhile(pred)`

Skips elements as long as the predicate `pred` holds. For example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .dropWhile(n => n < 3)
    .toArray(); // -> [3, 4, 5]
```

### `.filter(pred)`

Keeps only the elements matching the predicate `pred`. For example:
```ts
chainFrom([1, 2, 3, 4])
    .map(x => x % 2 === 1)
    .toArray(); // -> [1, 3]
```

### `.flatMap(f)`

For `f` a function which maps each element to an iterable, applies `f` to each
element and concatenates the results. For example:
```ts
const authors = [
    { name: "cbrontë", books: ["Jane Eyre", "Shirley"] },
    { name: "mshelley", books: ["Frankenstein"] },
]

chainFrom(authors)
    .mapcat(author => author.books)
    .toArray();
// -> ["Jane Eyre", "Shirley", "Frankenstein"]
```

### `.interpose(separator)`

Inserts `separator` between each pair of elements. For example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .interpose(0)
    .toArray();
// -> [1, 0, 2, 0, 3, 0, 4, 0, 5]
```

### `.keep(f)`

Transforms each element by applying `f` to it, but drops any elements for which
`f` returns `null` or `undefined` (but not other falsy values). For example:
```ts
chainFrom([{a: 1}, {a: 2}, {b: 3}])
    .keep(o => o.a)
    .toArray(); // -> [1, 2]
```

### `.map(f)`

Transforms each element by applying `f` to it. For example:
```ts
chainFrom([1, 2, 3])
    .map(x => x * 2)
    .toArray(); // -> [2, 4, 6]
```

### `.partitionAll(n)`

Groups elements into arrays of `n` elements. If the number of elements does not
divide perfectly by `n`, the last array will have fewer than `n` elements. For
example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .partitionAll(2)
    .toArray();
// -> [[1, 2], [3, 4], [5]]
```

### `.partitionBy(f)`

Groups consecutive elements for which `f` returns the same value (as determined
by `===`) into arrays. For example:
```ts
chainFrom(["a", "ab", "bc", "c", "cd", "cde"])
    .partitionBy(s => s[0])
    .toArray();
// -> [["a", "ab"], ["bc"], ["c", "cd", "cde"]]
```

### `.remove(pred)`

Like `filter()`, but removes the elements matching `pred` instead. For example:
```ts
chainFrom([1, 2, 3])
    .remove(x => x % 2 === 1)
    .toArray(); // -> [2, 4]
```

### `.take(n)`

Takes the first `n` elements and drops the rest. An essential opperation for
efficiency, because it stops computations from occurring on more elements of the
input than needed to produce `n` results. For example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .take(3)
    .toArray(); // -> [1, 2, 3]
```

### `.takeNth(n)`

Takes every `n`th element, starting from the first one. In other words, it takes
the elements whose indices are multiples of `n`. For example:
```ts
chainFrom([1, 2, 3, 4, 5, 6])
    .takeNth(2)
    .toArray(); // [1, 3, 5]
```

### `.takeWhile(pred)`

Takes elements as long as the predicate `pred` holds, then drops the rest. Like
`take()`, stops unnecessary computations on elements after `pred` fails. For
example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .takeWhile(n => n < 3)
    .toArray(); // -> [1, 2]
```

### `.compose(transducer)`

Add an arbitrary transducer to the chain. `transducer` should be a function
which implements the [transducer
protocol](https://githube.com/cognitect-labs/transducers-js#the-transducer-protocol),
meaning it is a function which takes a `Transformer` and returns another
`Transformer`. This is the most general transformation, and it is used by this
library internally to implement all the others. For example usage, see the
[Advanced Usage](#advanced-usage) section.

## Ending a chain

The following methods terminate a chain started with `chainFrom`, performing the
calculations and producing a result.

### `.count()`

Returns the number of elements. For example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .filter(x => x % 2 === 1)
    .count(); // -> 3
```

### `.every(pred)`

Returns `true` if all elements satisfy the predicate `pred`, or `false`
otherwise. Short-circuits computation once a failure is found. Note that this is
equivalent to `.remove(pred).isEmpty()`.

Example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .map(n => 10 * n)
    .every(n => n > 3); // -> true

chainFrom([1, 2, 3, 4, 5])
    .map(n => 10 * n)
    .every(n => n < 30); // -> false
```

### `.find(pred)`

Returns the first element of the result which satisfies the predicate `pred`, or
`null` if no such element exists. Note that this is equivalent to
`.filter(pred).first()`.

Example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .map(x => x * 10)
    .find(x => x % 6 === 0); // -> 30
```

### `.first()`

Returns the first element of the result, or `null` if there are no other
elements. Short-circuits computation, so no more work is done than necessary to
get the first element.

Example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .map(x => x * 10)
    .first(); // -> 10
```

### `.forEach(f)`

Calls `f` on each element of the result, presumably for side-effects. For
example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .map(x => x * 10)
    .forEach(x => console.log(x));
// Prints 10, 20, 30, 40, 50
```

### `.isEmpty()`

Returns `true` if there are any elements, else `false`. For example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .filter(n => n > 10)
    .isEmpty(); // -> true

chainFrom([1, 2, 3, 4, 5])
    .filter(n => n % 2 === 0)
    .isEmpty(); // -> false
```

### `.some(pred)`

Returns `true` if any element satisfies the predicate `pred`, or `false`
otherwise. Short-circuits computation once a match is found. Note that this is
equivalent to `.filter(pred).isEmpty() === false`.

Example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .map(n => 10 * n)
    .some(n => n === 30); // -> true

chainFrom([1, 2, 3, 4, 5])
            .map(n => 10 * n)
            .some(n => n === 1); // -> false
```


### `.stringJoin(separator)`

Returns a string obtained by concatenating the elements together as strings with
the separator between them. For example:
```ts
chainFrom([1, 2, 3, 4, 5])
    .filter(n => n % 2 === 1)
    .stringJoin(" -> "); // -> "1 -> 3 -> 5"
```
Not called `toString()` in order to avoid clashing with the `Object` prototype
method.

### `.toArray()`

Returns an array of the results. See any of the above examples.

### `.toIterator()`

Returns an iterator. Elements of the input iterator are not read until this
iterator is read, and then only as many as needed to compute number of results
requested. This is the primary way of reading results lazily.

Example:
```ts
const iterator = chainFrom([1, 2, 3, 4, 5])
    .map(x => x * 10)
    .toIterator();
console.log(iterator.next().value()); // Prints 10
// So far, the map function has only been called once.
```

### `.reduce(reducer, intialValue?)`

Reduces the elements according to the reducer, and returns the result. `reducer`
may be either a plain function of the form `(acc, x) => acc` or a transformer as
defined by the [transformer
protocol](https://github.com/cognitect-labs/transducers-js#transformer-protocol).
This is the most general way to terminate a chain, and all the others (except
for `toIterator`) are implemented using it.

Example of using a plain function reducer:
```ts
chainFrom([1, 2, 3, 4, 5])
    .map(x => x * 10)
    .reduce((acc, x) => acc + x, 0); // -> 150
```
A handful of pre-made transformers are provided by this library to be used with
`reduce()`. They are described in the next section.

## Reducers

These APIs provide objects which may be passed to `reduce()`, as described in
the previous section, to provide additional options for completing chains whose
elements are of specific types- for example, `toSum()` can only be used on a
chain of numbers.

They are kept as separate transformer objects, rather than added as additional
termination methods, to maintain type safety in TypeScript projects. TypeScript
does not provide a way to restrict which of an interface's methods may be called
depending on the value of a type parameter, so adding these as termination
methods would mean they could be called regardless of the current element type.
By contrast, passing one of these reducers to `reduce()` on a chain of the
incorrect type will be caught as an error by TypeScript.

### `toSum()`

For a chain of numbers, return their sum. For example:
```ts
chainFrom(["a", "bb", "ccc"])
    .map(s => s.length)
    .reduce(toSum()); // -> 6
```

### `toAverage()`

For a chain of numbers, return their average, or `NaN` if there are no elements.
For example:
```ts
chainFrom(["a", "bb", "ccc"])
    .map(s => s.length)
    .reduce(toAverage()); // -> 2
```

### `toMin(comparator?)`

Returns the minimum element, according to the comparator. If no comparator is
provided, then this reducer may only be applied if the elements are numbers and
uses the natural comparator. Returns `null` if there are no elements.

Example:
```ts
chainFrom(["a", "bb", "ccc"])
    .map(s => s.length)
    .reduce(toMin()); // -> 1
```

### `toMax(comparator?)`

Returns the maximum element, according to the comparator. If no comparator is
provided, then this reducer may only be applied if the elements are numbers and
uses the natural comparator. Returns `null` if there are no elements.

Example:
```ts
chainFrom(["a", "bb", "ccc"])
    .map(s => s.length)
    .reduce(toMax()); // -> 3
```

### `toObject()`

For a chain of two element arrays, creates an object whose keys are the first
element of each array and whose values are the second. The first element of each
pair must be a string. Further, a type parameter must be provided to specify the
type of the values. For example:
```ts
chainFrom(["a", "bb", "ccc"])
    .map(s => [s, s.length])
    .reduce(toObject<number>()); // -> { a: 1, bb: 2, ccc: 3 }
```

## Utility functions

### `makeTransducer(f: (reducer, result, input) => result)`

Convenience function for defining new transducers. Given a function which takes
a reducer and returns another reducer, return a transducer.

The full definition of a transducer is a function from transformer to
transformer, but most transducers can more easily be thought of as a function
from reducer to reducer instead. This helper allows transducers to be defined in
terms of such a function, removing the boilerplate of defining
`@@transducer/init`, etc. methods on a transformer.

Note that the signature `reducer => reducer` expands to `reducer => (result,
input) => result`, which after uncurrying becomes `(reducer, result, input) =>
result`, the actual type in the signature of this function.

An example is given in the [Advanced
Usage](https://github.com/dphilipson/typescript-transducers#using-custom-transducers)
section of the readme, where
```ts
function replace<T>(initial: T, replacement: T) {
    return makeTransducer((reducer, result, input) => {
        const output = intput === initial ? replacement : input;
        return reducer(result, output);
    });
}
```
is equivalent to
```ts
import {
    CompletingTransformer,
    Transducer,
    Transformer,
 } from "typescript-transducers";

function replace<T>(initial: T, replacement: T): Transducer<T, T> {
    return (xf: CompletingTransformer<T, any, T>) => ({
        ["@@transducer/init"]: () => xf["@@transducer/init"](),
        ["@@transducer/result"]: (result: T) => xf["@@transducer/result"](result),
        ["@@transducer/step"]: (result: T, input: T) => {
            const output = input === initial ? replacement : input;
            return xf["@@transducer/step"](result, output);
        },
    });
}
```

### `reduced(result)`

Returns a reduced value of `result`, as described by the [transducer
protocol](https://github.com/cognitect-labs/transducers-js#reduced). Can be
returned by a reducer or a transformer to short-circuit computation.

Copyright © 2017 David Philipson