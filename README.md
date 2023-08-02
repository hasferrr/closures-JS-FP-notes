# Closures in JavaScript vs Closures in FP

Closures in JavaScript is different from closures in other purely functional programming languages like Haskell, SML, OCaml, etc.

- [Terminology](#terminology)
- [Function that returns an Anonymous Function](#function-that-returns-a-function)
- [What happened if you 'redefine' variable `name`, after define variable `call`?](#what-happened-if-you-redefine-variable-name-after-define-variable-call)
- [Mutate before define `call` function](#mutate-before-define-call-function)
- [Correct way to create a closure in JavaScript](#correct-way-to-create-a-closure-in-javascript)
- [Other (Closure with Object)](#other-closure-with-object)
- [Other (Array)](#other-array)

## Terminology

**Environment**

- Environment is sequence of bindings (like variable bindings, function bindings, etc)
- For Example: in JavaScript when you write `let x = 1`, it stores `x` as a binding that maps / references the primitive value of `1` within the environment

**Mutation**

- Mutation (mutate) is the process of altering the value of a variable in the environment to a new value.

**Shadowing**

- In Functional Programming, there is no way to change the contents of a binding (because it's immutable).
- Once a variable is assigned a value, it will forever map to that value in the environment. No assignment statement that lets you change the value of a variable in the environment.
- The variable *shadows* when you create new variable with the same name you previously defined.
- It will creates a different environment where the later binding for the variable *shadows* the earlier one.
- And the rest of the program will use this new environment (with the new Variable and Value you lastly defined (but the Old Variable and its Old Value is still exist)).

**Free variable**

- A free variable is a variable used in a function that is not defined within that function. Instead, it is defined in an outer scope.

## Function that returns a Function

JavaScript code:

```js
let name = 'AAAAAAAAAAAAA'

const sayHello = () => {
  const text = 'Hello, ' + name
  return () => console.log(text)
}

const call = sayHello()
call()  // 'Hello AAAAAAAAAAAAA'
```

- Inside the `sayHello` function, the local variable `text` will look for the `name` variable and map whatever the `name` variable is mapping to, instead of copying the `name` variable.
- because `text` uses free variable `name`
- `text` will be **'Hello, AAAAAAAAAAAAA'**
- `sayHello` function returns the function that logs the `text`
- In global scope, we create variable `call` (which is a function returned from `sayHello`)
- When `call` is executed, it will logs **'Hello AAAAAAAAAA'**
- Easy

## What happens  if you 'redefine' variable `name`, after defining variable `call`?

#### JavaScript code:

```js
let name = 'AAAAAAAAAAAAA'

const sayHello = () => {
  const text = 'Hello, ' + name
  return () => console.log(text)
}

const call = sayHello()
call()      // 'Hello AAAAAAAAAAAAA'

name = 'BBBBBBBBBBBB'                 // <---------- Mutate Here
call()      // 'Hello BBBBBBBBBBBBB'
```

- `call` is not closures
- closures in the two examples above are WRONG in JavaScript
- `call()` will log whatever the new value of variable `name` is
- Unlike purely functional programming, in JavaScript, when you redefine variable `name`, it will **mutate** that variable, and the `name` variable maps into new value.
- It's look like the scope is 'more dynamic'?

#### Functional Programming code:

```js
name = 'AAAAAAAAAAAAA'            // old name

function sayHello()
  text = 'Hello, ' + name
  return () => console.log(text)

call = sayHello()
call()  // 'Hello AAAAAAAAAAAAA'

name = 'BBBBBBBBBBBB'             // new name shadows the old name
call()  // 'Hello AAAAAAAAAAAAA'
```

- In Functional Programming like Haskell, SML, OCaml, etc., redefining variable `name` it will not mutate the defined variable
- Instead, it will **shadows** the variable `name` (creating a new variable with the same name that maps/references to the new value)
- Shadowing doesn't mutate the old variable. Rather, in the environment, it will create new variable with the same name and maps into the new value.
- The old `name` still exist in the environment, which is still maps into **'AAAAAAAAAAA'**
- The new `name` is created and 'shadows' the old `name`, which is maps into **'BBBBBBBBBBBB'**
- This is why, in purely Functional Programming, when you create function like `sayHello`, it will **close** (closure) the environment and the function will look up the Free Variable (like the variable `name`) when the function was defined, not when the function is called
- As a result, it will use the old name, which is different from JavaScript.

## Mutate before define `call` function

```js
let name = 'AAAAAAAAAAAAA'

const sayHello = () => {
  const text = 'Hello, ' + name
  return () => console.log(text)
}

name = 'BBBBBBBBBBBB'       // <--------- Mutate Here
const call = sayHello()
call()                      // 'Hello BBBBBBBBBBBB'
```

- as you expected
- it will logs 'Hello BBBBBBBBBBBB' because you mutate it

## Correct way to create a closure in JavaScript

```js
let name = 'AAAAAAAAAAAAA'

const sayHello = (name) => {        // <------ don't use Free Variable,
                                    // instead copy it (as a function parameter)
  const text = 'Hello, ' + name
  return () => console.log(text)
}

const call = sayHello(name)        // <------- store the variable as an argument
call()  // 'Hello AAAAAAAAAAAAA'

name = 'BBBBBBBBBBBB'              // <------- Mutate global variable
call()  // 'Hello AAAAAAAAAAAAA'
```

In this way:

- `call()` will log 'Hello AAAAAAAAAAAAA'
- Why? Because `sayHello` function **closes** the environment
- Because `sayHello` function use the local variable `name` which is copy of the gobal variable `name` (except you store an Object)
- This is closures in JavaScript

## Other (Closure with Object)

### Closure with Object (1)

```js
let name = {qqq: 'OOBBBJJJEEECTTTT'}

const sayHello = (name) => {
  return () => console.log('Hello', name)
}

const call = sayHello(name)
call()    // Hello {qqq: 'OOBBBJJJEEECTTTT'}

name.qqq = 'NEW'
call()    // Hello {qqq: 'NEW'}
```

- JavaScript dont copy an Object (or Array) as a function parameter!!!
- make a copy yourself!


### Closure with Object (2)

```js
let name = {qqq: 'OOBBBJJJEEECTTTT'}

const sayHello = (name) => {
  return () => console.log('Hello', name)
}

const call = sayHello(name)
call()    // Hello {qqq: 'OOBBBJJJEEECTTTT'}

name = {new: 'NEW VALUE'}
call()    // Hello {qqq: 'OOBBBJJJEEECTTTT'}
```

- still reference / maps to garbage!!!, I mean **old object** forever!!!!


## Other (Array)

If you dont copy the array:

```js
let name0 = [7,8,9]
let name = name0        // references to the same Array

const sayHello = () => {
  return () => console.log(name)
}

const call = sayHello()
call()                  // [ 7, 8, 9 ]

name0[0] = 777
call()                  // [ 777, 8, 9 ]
```

```js
let name = [7,8,9]

const sayHello = (name) => { // still references to the same Array
  return () => console.log(name)
}

const call = sayHello(name)
call()                  // [ 7, 8, 9 ]

name[0] = 777
call()                  // [ 777, 8, 9 ]
```

If you copy the array:

```js
let name = [7,8,9]
let copy = [...name]    // COPY the Array

const sayHello = () => {
  return () => console.log(copy)
}

const call = sayHello()
call()                  // [ 7, 8, 9 ]

name[0] = 777
call()                  // [ 7, 8, 9 ]
```

```js
let name = [7,8,9]

const sayHello = (name) => {
  const copy = [...name]    // COPY the Array
  return () => console.log(copy)
}

const call = sayHello(name)
call()                  // [ 7, 8, 9 ]

name[0] = 777
call()                  // [ 7, 8, 9 ]
```
