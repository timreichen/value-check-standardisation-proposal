# Value Check Standardisation Proposal

## Synopsis
Native sugar methods to standardise value checking in JavaScript.

## Motivation
JavaScript has some very unintuitive ways to check types and values.

### ```instanceof``` works great
```ìnstanceof``` works great to check the type of an object

```js
const date = new Date()
date instanceof Date // true
```
What about checking primitives who are not an instance? What to do if you want to know if a value is a object and not a subclass?
```js
const date = new Date()
date instanceof Object // true
```
I think JavaScript can do better than expecting the developer to know  ```value.constructor.name === "Object"```

### ```typeof``` is unintuitive
as explored in [this](https://charlieharvey.org.uk/page/javascript_the_weird_parts) article, typeof can behave unexpected for values.
Who would have thought that ```NaN```, an acronym for *Not A Number*, returns
```js
typeof NaN // "number"
```
This unintuitive behavior was noticed and since ```NaN !== NaN // true```(?!?) is as unexpected as it gets a method was implemented:
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

Some classes do not have the accoring method at all:
```js
Boolean.isBoolean(true) // TypeError: Boolean.isBoolean is not a function.
String.isString("foo") // TypeError: String.isString is not a function.
```

## Proposal
Classes are extended with static methods according to the table below.
These methods are standardised to compare primitive values as well as object instances.

| Method | Description | Implemented | Behavior |
|---|---|---|---|
| Boolean.isBoolean | checks if value is a boolean primitive or a ```Boolean``` instance || ```Boolean.isBoolean = value => Object.prototype.toString.call(value) === "[object Boolean]"``` |
| Number.isNumber | checks if value is an number primitive or a ```Number``` instance | | ```Number.isNumber = value => Object.prototype.toString.call(value) === "[object Number]" && !Number.isNaN(value) && isFinite(value)``` |
| Number.isInteger | checks if value is an integer | (:heavy_check_mark:)* |   |
| Number.isFloat | checks if value is a float | | ```Number.isFloat = value => Number.isNumber(value) && !Number.isInteger(value)```* |
| Number.isNaN | checks if value is ```NaN``` | :heavy_check_mark: |
| Number.isFinite | checks if value is finite (not ```Infinity```) | :heavy_check_mark: |
| String.isString | checks if value is a string primitive or a ```String``` instance | | ```String.isString = value => Object.prototype.toString.call(value) === "[object String]" ``` |
| Object.isObject | checks if value is an ```Object``` instance | | ```Object.isObject = value => Object.prototype.toString.call(value) === "[object Object]"``` |
| Symbol.isSymbol | checks if value is a ```Symbol``` | | ```Symbol.isSymbol = value => Object.prototype.toString.call(value) === "[object Symbol]"``` |
| Array.isArray | checks if value is an ```Array```instance | :heavy_check_mark: |


*extend so ```Number``` instances are treated as follows:
```js
Number.isInteger(new Number(1)) // true
```
