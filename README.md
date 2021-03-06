# JS Concepts Gotcha

## About

JS concepts demonstrated by code, mostly it follows the contents of [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/)

## Quick links

1. [IIFE: Immediately Invoked Function Expression](#IIFE)
1. [Scope](#Scope)
1. [Hoisting](#Hoisting)
1. [Closure](#Closure)
1. [Modules](#Modules)
1. [Determining `this`](#determining-this)

# <a id="IIFE"></a>IIFE: Immediately Invoked Function Expression

## Basic

```js
(function IIFE() {
    // ...
})();
```

or

```js
(function () {
    // ...
})();
```

## Function as parameter

```js
(function (def) {
    def(window);
})(function def(global) {
    // ...
});
```

## Declaration execution syntax error

```js
function () {}() // SyntaxError
```

## Handle syntax error

### Convert declaration to expression

```js
!function () {}() // ! can be replaced with +, -, ~
```

### Alternative

```js
(function () {})();
```

**[⬆ back to top](#quick-links)**

# <a id="Scope"></a>Scope

## Function scope

```js
function foo() {
    var a = 1;
    console.log(a); // 1
}

console.log(a); // undefined
```

## Block scopes

```js
for (; ;) {
    // block scope
}
```

```js
if (true) {
    // block scope
}
```

```js
with (window) {
    // block scope
}
```

```js
try {
    // block scope
}
catch (err) {
    // block scope
}
```

## Explicit block scope

```js
{
    // explicit block scope
}
```

## Block scopes pollution

```js
for (var i = 0; i < 5; i++) {
    // ...
}

console.log(i); // 5
```

```js
if (true) {
    var a = 1;
}

console.log(a); // 1
```

## Block scopes protection

### ES6

```js
for (let i = 0; i < 5; i++) {
    // ...
}

console.log(i); // ReferenceError
```

```js
if (true) {
    let a = 1;
}

console.log(a); // ReferenceError
```

```js
{
    let a = 1;
}

console.log(a); // ReferenceError
```

### pre-ES6

```js
try {
    throw undefined;
} catch (a) {
    a = 1;
}

console.log(a); // ReferenceError
```

## Cheating scope

```js
function foo() {
    console.log(a); // undefined
    var a;
}

function bar() {
    console.log(a); // ReferenceError
    eval('var a;');
}

foo();
bar();
```

**[⬆ back to top](#quick-links)**

# <a id="Hoisting"></a>Hoisting

## Variable hoisting

```js
a = 1;

var a;

console.log(a); // 1
```

## Function hoisting

```js
foo(); // here

function foo() {
    console.log('here');
}
```

## Declarations hoisting & assignments stay

### Variable assignment

```js
console.log(b); // ReferenceError
console.log(a); // undefined

var a = 1;

console.log(a); // 1
```

### Function assignment

```js
console.log(bar); // ReferenceError
console.log(foo); // undefined

var foo = function bar() {
    // ...
};
```

**[⬆ back to top](#quick-links)**

# <a id="Closure"></a>Closure

```js
function foo() {
    // bar's closure starts
    
    var a = 1;

    function bar() {
        console.log(a);
    }

    return bar;
    
    // bar's closure ends
}

var baz = foo();
baz(); // 1
```

**[⬆ back to top](#quick-links)**

# <a id="Modules"></a>Modules

## Two requirements

```js
// 1. an outer enclosing function to be invoked
function foo() {
    var a = 1;

    function bar() {
        console.log(a);
    }

    // 2. return back at least one inner function
    return {
        bar: bar
    };
}

var baz = foo();

baz.bar(); // 1
```

## Singleton

```js
var baz = (function foo() {
    var a = 1;

    function bar() {
        console.log(a);
    }

    return {
        bar: bar
    };
})();

baz.bar(); // 1
```

## publicAPI

```js
var foo = (function () {
    function bar() {
        console.log('bar');
    }
    
    function baz() {
        console.log('baz');
    }
    
    var publicAPI = {
        bar: bar
    };

    return publicAPI;
})();

foo.bar(); // bar
foo.baz(); // TypeError
```

## Modern Modules

### Manager

```js
var module = (function () {
    var modules = {};

    function define(name, depNames, fn) {
        for (var i = 0, deps = []; i < depNames.length; i++) {
            deps[i] = modules[depNames[i]];
        }
        
        modules[name] = fn.apply(fn, deps);
    }

    function get(name) {
        return modules[name];
    }

    return {
        define: define,
        get: get
    };
})();
```

### define

```js
module.define('dep', [], function () {
    var name = 'dep';
    
    function baz () {
        return name;
    }

    return {
        baz: baz
    };
});

module.define('foo', ['dep'], function (dep) {
    var name = 'foo';

    function bar() {
        return name;
    }

    function qux() {
        console.log(bar(), dep.baz());
    }
    
    return {
        qux: qux
    };
});
```

### get

```js
var foo = module.get('foo');

foo.qux(); // foo, dep
```

## Future Modules (ES6)

**dep.js**

```js
var name = 'dep';

function baz() {
    return name;
}

export dep;
```

**foo.js**

```js
import baz from 'dep';

var name = 'foo';

function bar() {
    return name;
}

function qux() {
    console.log(bar(), dep.baz());
}

export foo;
```

```js
import foo from 'foo';

foo.qux(); // foo, dep
```

**[⬆ back to top](#quick-links)**


# Determining `this`

## Default binding

```js
function foo() {
    console.log(this);
}

foo(); // global
```

### Strict mode
```js
function foo() {
    "use strict";

    console.log(this);
}

foo(); // undefined
```

```js
function foo() {
    console.log(this);
}

(function(){
    "use strict";

    foo(); // global
})();
```

## Implicit binding

```js
function foo() {
    console.log(this);
}

var obj = {
    foo: foo
};

obj.foo(); // obj
```

### Implicit binding lost

```js
function foo() {
    console.log(this);
}

var obj = {
    foo: foo
};

var bar = obj.foo;

bar(); // global
```

```js
function foo() {
    console.log(this);
}

var obj = {
    foo: foo
};

setTimeout(obj.foo, 100);   // global

setTimeout(function() {
    obj.foo();              // obj
}, 200);
```

## Explicit binding

### `call` and `apply`

```js
function foo() {
    console.log(this);
}

var obj = {};

foo();          // global
foo.call(obj);  // obj
foo.apply(obj); // obj
```

### Hard binding: `Function.prototype.bind`

```js
function foo() {
    console.log(this);
}

var obj = {};
var bar = foo.bind(obj);

bar(); // obj
```

### API call contexts

```js
function foo() {
    console.log(this);
}

var obj = {};

[1, 2, 3].forEach(foo, obj); // obj, obj, obj
```

## `new` binding

```js
function foo(a) {
    this.a = a;
}

var bar = new foo(1); 
console.log(bar.a); // 1
```

## Binding exceptions

### Ignored `this`

```js
function foo() {
    console.log(this);
}

foo.apply(null); // global
```

To spread parameters:

#### ES5

```js
function foo(a, b) {
    console.log(a, b);
}

foo.apply(null, [1, 2]); // 1, 2
```

#### ES6

```js
function foo(a, b) {
    console.log(a, b);
}

foo(...[1, 2])
```

### Safer `this` by DMZ: de-militarized zone

```js
function foo(a, b) {
    console.log(a, b);
}

var ø = Object.create(null);

foo.apply(ø, [1, 2]); // 1, 2
```

### Indirect references

```js
function foo() {
    console.log(this);
}

var obj = {};

(obj.foo = foo)();    // global
obj.foo();            // obj
```

## ES6 arrow function `this` binding adopt

```js
var foo = {
    baz: () => {
        console.log(this);
    }
}

var bar = {
    baz: function () {
        console.log(this);
    }
}

foo.baz(); // global
bar.baz(); // bar
```

**[⬆ back to top](#quick-links)**

# Objects

## Access via property & key
```js
var foo = {};

foo.bar = 1;        // property
foo['baz'] = 1;     // key
```

## Access via invalid identifier key
```js
var foo = {};

foo['b-a-r!'] = 1;  // key (invalid identifier)
```

## Auto convert key type
```js
var foo = {};
var bar = {};

foo[true] = 1;
foo[3] = 2;
foo[bar] = 3;

console.log(foo['true']);               // 1
console.log(foo['3']);                  // 2
console.log(foo['[object Object]']);    // 3
```

## Array

### Add number property
```js
var arr = ['foo', 'foo'];

arr['1'] = 'bar';

console.log(arr[1]);     // bar
console.log(arr['1']);   // bar

## Duplicating Objects

### JSON-safe copy
```js
var newObj = JSON.parse(JSON.stringify(someObj));
```

### ES6 shallow copy: `Object.assign`
```js
var anotherObj = {
    foo: 'bar'
};

var oldObj = {
    a: 1,
    b: anotherObj
};

var newObj = Object.assign({}, oldObj);

console.log(newObj.a);                       // 1
console.log(newObj.b === anotherObject);     // true
```

## Property descriptors

### Writable
```js
var obj = {};

Object.defineProperty(obj, 'a', {
    value: 1,
    writable: false
});

obj.a = 2;

console.log(obj.a); // 1 | TypeError in strict mode
```

### Configurable
```js
var obj = {};

Object.defineProperty(obj, 'a', {
    value: 1,
    configurable: false
});

delete obj.a;       // failed silently

console.log(obj.a); // 1

Object.defineProperty(obj, 'a', {
    value: 2
}); // TypeError
```

### Enumerable
```js
var obj = {};

Object.defineProperty(obj, 'a',{
    value: 1,
    enumerable: false
});

for (var k in obj) {
    console.log(k); // a not logged
}
```

## Immutability

### Object Constant
```js
var obj = {};

Object.defineProperty(obj, "CONST_VARIABLE", {
    value: 1,
    writable: false,
    configurable: false
});

obj.CONST_VARIABLE = 2;             // TypeError in strict mode

console.log(obj.CONST_VARIABLE);    // 1
```

### Prevent extensions
```js
var obj = {
    a: 2
};

Object.preventExtensions(obj);

obj.b = 3;
console.log(obj.b); // undefined
```

### Seal
```js
var obj = {
    a: 1
};

Object.seal(obj);

obj.a = 2;          // can change
obj.b = 1;          // can't add
delete(obj.a);      // can't delete

console.log(obj.a); // 2
console.log(obj.b); // undefined
```

### Freeze
```js
var obj = {
    a: 1
};

Object.freeze(obj);

obj.a = 2;          // can't change
obj.b = 1;          // can't add
delete(obj.a);      // can't delete

console.log(obj.a); // 1
console.log(obj.b); // undefined
```

## Existence

### `in` and `hasOwnProperty`
```js
var obj = {
    a: 1
};

console.log('a' in obj);                // true
console.log(obj.hasOwnProperty('a');    // true
```

### Enumeration
```js
var obj = {};

Object.defineProperty(obj, 'a',{
    value: 1,
    enumerable: false
});

console.log(obj.a);                             // 1
console.log('a' in obj);                        // true
console.log(obj.hasOwnProperty('a'));           // true

for (var k in obj) {
    console.log(k); // 'a' not logged
}

console.log(Object.keys(obj));                  // []
console.log(Object.getOwnPropertyNames(obj));   // ['a']
```
