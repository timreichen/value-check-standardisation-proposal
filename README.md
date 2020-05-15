# Value Check Standardisation Proposal

## Synopsis
Native sugar methods to standardise value checking in JavaScript.

## Motivation
JavaScript has some very unintuitive ways to check types and values.

### ```instanceof``` works, but not for iframes
```ìnstanceof``` works great to check the type of an object

```js
const date = new Date()
date instanceof Date // true
```
It fails when using an iFrame as described [here] (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray). Therefore an extra method ```Array.isArray``` was implemented.

And what about checking primitives who are not an instance?

### ```typeof``` is not always intuitive
as explored in [this](https://charlieharvey.org.uk/page/javascript_the_weird_parts) article, ```typeof``` can behave unexpected for certain values.
Who would have thought that ```NaN```, an acronym for *Not A Number*, returns
```js
typeof NaN // "number"
```
This unintuitive behavior was noticed and since ```NaN !== NaN // true``` (?!?) is as unexpected as it gets a method was implemented:
```js
Number.isNaN(NaN) // true (window.isNaN also exists)
```
## Inconsistent behavior
Over the years additional classes were extended to make type and value checking easier but also behaves inconsistent:

Only evaluates primitive
```js
Number.isInteger(1) // true
Number.isInteger(new Number(1)) // false
```
evaluates primitive and object
```js

Number.isNaN(NaN) // true
Number.isNaN(new Number(NaN)) // true

Array.isArray([]) // true
Array.isArray(new Array()) // true
```

Some classes do not have the accoring and expected methods at all:
```js
Boolean.isBoolean(true) // TypeError: Boolean.isBoolean is not a function.
String.isString("foo") // TypeError: String.isString is not a function.

Number.isFloat(1) // TypeError: Number.isFloat is not a function.
```

## Problems
- ```instanceof``` and ```typeof``` are useful but not always intuitive. Also ```instanceof``` is not always reliable.
- Javascript has implemented methods to help value check but their behavior is inconsistent
- Some useful and expected methods are missing completely.


## Proposal
- Extended core type classes with static methods according to the table below.
- Make their behavior consistent

| Method | Description | Implemented | Behavior |
|---|---|---|---|
| Boolean.isBoolean | checks if value is a boolean primitive or a ```Boolean``` instance || ```Boolean.isBoolean = value => Object.prototype.toString.call(value) === "[object Boolean]"``` |
| Number.isNumber | checks if value is an number primitive or a ```Number``` instance | | ```Number.isNumber = value => Object.prototype.toString.call(value) === "[object Number]" && !Number.isNaN(value) && isFinite(value)``` |
| Number.isInteger | checks if value is an integer | (:heavy_check_mark:)* |   |
| Number.isFloat | checks if value is a float | | ```Number.isFloat = value => Number.isNumber(value) && !Number.isInteger(value)```* |
| Number.isNaN | checks if value is ```NaN``` | :heavy_check_mark: |
| Number.isFinite | checks if value is finite | :heavy_check_mark: |
| String.isString | checks if value is a string primitive or a ```String``` instance | | ```String.isString = value => Object.prototype.toString.call(value) === "[object String]" ``` |
| Object.isObject | checks if value is an ```Object``` instance | | ```Object.isObject = value => Object.prototype.toString.call(value) === "[object Object]"``` |
| Symbol.isSymbol | checks if value is a ```Symbol``` | | ```Symbol.isSymbol = value => Object.prototype.toString.call(value) === "[object Symbol]"``` |
| Array.isArray | checks if value is an ```Array```instance | :heavy_check_mark: |


*extend so ```Number``` instances are treated as follows:
```js
Number.isInteger(new Number(1)) // true
```
