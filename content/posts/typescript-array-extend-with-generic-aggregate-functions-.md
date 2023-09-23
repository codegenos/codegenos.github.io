---
author: "CodeGenos"
title: "Typescript: Extend Array type with sum count avg min max Aggregates"
date: 2023-09-03T21:20:16+03:00
description: "Add generic aggregate functions like count, sum, average, min and max to Arrays in Typescript"
tags: ["Typescript", "Javascript", "prototype"]
categories: ["Typescript", "Javascript"]
ShowToc: false
ShowBreadCrumbs: true
draft: false
---

In this article i'll show you how to implement `sum`, `count`, `avg`, `min` and `max` aggregate functions that applies to all Array instances in typescript.

First we will learn how to extend Array type:

## Extend Array prototype
In javascript, you can extend an existing class by adding a new function to its prototype without modifying its source code. 

To add new function to `Array` type, you basically set the new function to `Array.prototype.<functionName>` and you can use the new function for all the instances of the class you create.

In typescript, if you want to extend an existing type with a new function, you must add an interface declaration which contains the new function declaration. Otherwise, when you try to extend the prototype with the new function name, you will get an `Property 'functionName' does not exist on type 'any[]'` error. 


Here is the example how to add a new function `functionName` to `Array`.

```typescript
// If you export nothing, you'll need this. Otherwise, delete it 
export {}

declare global {
    // existing declarations in the global namespace
    interface Array<T> {
         // add new function
         functionName()
     }
 }
 
 // set the prototype
 Array.prototype.functionName = function() {
    // implementation
 }

 // create new Array instance
 const arr = []

 // you can call the new added functionName
 arr.functionName()
```

- Since `Array` type is in the `global` namespace, we define `Array<T>` interface in the global namespace. We put the interface in `declare global { }`
- We add the `functionName` function to both **interface** and **prototype**.

## Sum
Now that we have learned how to extend the Array, let's implement the `sum` function. 
I used `reduce` function to calculate sum. You can also use `forEach` function or `for` loop.

```typescript
export {}

declare global {
    interface Array<T> {
        sum(): number
    }
}

Array.prototype.sum = function<T extends number>(): number {
    return (this as Array<T>).reduce((prev, current) => prev + current, 0)
}

const arr = [1, 2, 3, 4]

console.log(arr.sum())
```

- Since `sum` function only supports `number` we make generic type `<T extends number>`.

To support arrays with more complex objects you can add map function `mapFn: (v: T) => number`. Then you can use `sum` function like: `arr.sum(x => x.price)`. The `avg`, `min` and `max` implementations will use map function.

```typescript
export {}

declare global {
    interface Array<T> {
        sum(mapFn: (v: T) => number): number
    }
}

Array.prototype.sum = function<T>(mapFn: (v: T) => number): number {
    return (this as Array<T>).map(v => mapFn(v)).reduce((prev, current) => prev + current, 0)
}

interface OrderItem {
    price: number
}

const arr: OrderItem[] = [{ price: 50 }, { price: 100 }]

console.log(arr.sum(x => x.price))
```

You can also combine with `filter` function:

```typescript
arr.filter(x => x.price > 50).sum(x => x.price)
```

- In this example, array items with a **price bigger than 50** are collected.

## Count

The `count` function simply returns `length` of the array.

```typescript
Array.prototype.count = function<T>(): number {
    return (this as Array<T>).length
}
```

Usage:

```typescript
arr.count()
```

## Average

The `avg` function returns `sum / count` of the array.

For arrays with 0 count, it will return 0.

```typescript
Array.prototype.avg = function<T>(mapFn: (v: T) => number): number {
    const arr = this as Array<T>
    const count = arr.count()
    return count === 0 ? 0 : arr.sum(mapFn) / count
}
```

Usage:

```typescript
arr.avg(x => x.price)
```

## Min

The `min` function uses `Math.min` function.

Singe `Math.min` returns `Infinity` for empty arrays, `min` function returns `Infinity` for empty arrays.

```typescript
Array.prototype.min = function<T>(mapFn: (v: T) => number): number {
    return Math.min(...(this as Array<T>).map(v => mapFn(v)))
}
```

## Max

The `max` function uses `Math.max` function.

Singe `Math.max` returns `-Infinity` for empty arrays, `max` function returns `-Infinity` for empty arrays.


```typescript
Array.prototype.max = function<T>(mapFn: (v: T) => number): number {
    return Math.max(...(this as Array<T>).map(v => mapFn(v)))
}
```

## Conclusion

The overall code (`aggregates.ts`) looks like this:

```typescript
export {}

declare global {
    interface Array<T> {
        sum(mapFn: (v: T) => number): number

        avg(mapFn: (v: T) => number): number

        min(mapFn: (v: T) => number): number

        max(mapFn: (v: T) => number): number

        count(): number
    }
}

Array.prototype.sum = function<T>(mapFn: (v: T) => number): number {
    return (this as Array<T>).map(v => mapFn(v)).reduce((prev: number, current: number): number => prev + current, 0)
}

Array.prototype.count = function<T>(): number {
    return (this as Array<T>).length
}

Array.prototype.avg = function<T>(mapFn: (v: T) => number): number {
    const arr = this as Array<T>
    const count = arr.count()
    return count === 0 ? 0 : arr.sum(mapFn) / count
}

Array.prototype.min = function<T>(mapFn: (v: T) => number): number {
    return Math.min(...(this as Array<T>).map(v => mapFn(v)))
}

Array.prototype.max = function<T>(mapFn: (v: T) => number): number {
    return Math.max(...(this as Array<T>).map(v => mapFn(v)))
}
```

To use these aggregate functions in your code, you can import the module like this:

```typescript
import './aggregates'
```

Thanks for reading.

Happy coding!