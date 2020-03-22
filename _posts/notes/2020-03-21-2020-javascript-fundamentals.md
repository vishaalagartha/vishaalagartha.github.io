---
title: "JavaScript Fundamentals"
date: 2020-03-21
permalink: /notes/2020/03/21/javascript-fundamentals
tags:
--- 

# Passing Function Parameters

- Primitive types (strings, numbers, booleans, null, and undefined) are pass by value - modifications within a function will not be reflected outside of function
- Complex types (arrays and objects) are pass by reference - modifications within a function will be reflected outside of function

{% highlight javascript %}

function f1(a){
  a = 1
  return a
}

const a = 0
console.log(f1(a)) // 1

function f2(arr){
  arr[0] = 1
  return arr
}

const arr = [0, 0]
console.log(f2(arr)) // [0, 1]

{% endhighlight %}

# `const` vs. `let` vs. `var`

## Declaration
`var` can be redeclared as many times as one wants. `let` and `const` can only be declared once

{% highlight javascript %}
var x = 10
var x = 9

let y = 10
let y = 9 // Error

const z = 10
const z = 9 // Error
{% endhighlight %}

## Assignment

`var` and `let` can be reassigned. `const` cannot be reassigned.
NOTE: `const` arrays and objects can be modified, but cannot change the ENTIRE array or object
{% highlight javascript %}
var x = 10
x = 9

let y = 10
y = 9

const z = 10
z = 9 // Error
const arr = [0, 0]
arr[0] = 1 // Ok
arr = [1, 1] // Error
{% endhighlight %}

## Scope
`var` has function scope - it exists within the function it is defined in
`let` and `const` have block scope - it exists within the { } it is defined in
{% highlight javascript %}
{
  let i = 0
  var j = 0
  const k = 0
}
console.log(j)
console.log(j) // Error
console.log(k) // Error
{% endhighlight %}

## Hoisting
`var` is hoisted, but initialized as `undefined`. `const` and `let` are not - leading to ReferenceErrors
{% highlight javascript %}
console.log(i) // undefined
console.log(j) // Error
console.log(k) // Error

var i = 0
console.log(i) // 0
let j = 0
const k = 0
{% endhighlight %}

# `this`

`this` inside functions/constructors is the global scope

`this` inside methods refers to the object (implicit binding)


{% highlight javascript %}
const a = {
  b: 1,
  f(){
    console.log(this.c)
    console.log(this.b)
  },
  c: 0
}
a.f()
// 0
// 1
{% endhighlight %}

`this` with `call`, `apply`, or `bind` resets what `this` is (explicit binding).

Note that `bind` does not actually call the function, it simply binds it and creates a NEW function to be called in the future.

{% highlight javascript %}
function f(){
  console.log(this.a)
}

const o = {a: 'hello'}
f.apply(o) // hello
f.call(o) // hello

const newF = f.bind(o) 
newF() // hello
{% endhighlight %}

`this` in arrow functions inherit the parent context

{% highlight javascript %}
const a = {
  b: 1,
  f: () => {
    console.log(this.b)
  },
  g: function(){
    console.log(this.b)
  }
}
a.f() // undefined
a.g() // 1

{% endhighlight %}

# Promises 

## Basics

{% highlight javascript %}
const p = new Promise((resolve, reject) => {
  resolve('resolved')
  reject('rejected')
})

p.then((data) => console.log(data))
  .catch((error) => console.log(error))
{% endhighlight %}

# Closures

Functions that have access to their outer function scope even after returning. Hence, a closure can remember and access variables **even after the function has returned**.


{% highlight javascript %}
function f() {
  let x = 0
  return () => x++
}

let y = f() // f has returned, but x STILL EXISTS

console.log(y()) // 0
console.log(y()) // 1
console.log(y()) // 2
{% endhighlight %}
