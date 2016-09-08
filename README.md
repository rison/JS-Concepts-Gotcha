# JS Concepts Gotcha 

# IIFE: Immediately Invoked Function Expression

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

# Scope

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

# Hoisting

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


# Closure

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

# Modules

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

```
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
