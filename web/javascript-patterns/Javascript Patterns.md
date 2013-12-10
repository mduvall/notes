# Chapter 2: Essentials

## Side effects when forgetting var

- Globals without `var` cannot be deleted
- Implied globals in function scope without `var` can be deleted

```javascript
var global_var = 1;
global_novar = 2;

(function() {
  global_fromfunc = 3;
}());

delete global_var; // false
delete global_novar; // true
delete global_fromfunc; // true
```

## Single var pattern

Useful to put all var statements at the top of your function:

- Single place to look
- Prevents usage before definition
- Standard to avoid accidentally declaring globals

Looks like:

```javascript
function() {
  var a = 1,
      b = 2,
      c;
}
```

## Hoisting: scattering vars

Javascript takes all variable declarations in a function scope and puts the declaration at the top for you.

```javascript
myname = "foo";

function f() {
  alert(myname); // "undefined"
  var myname = "foo";
  alert(myname); // "local"
}

f();
```

## For loops

Avoid getting the `length` property multiple times for array-like objects (`arguments` and `HTMLCollection`s),
if the list is alive it is a costly operation. Cache the value.

```javascript
for (var i = 0, len = arguments.length; i < len; i++) {}

```

## For-in loops

Iterating over an object using `for (x in obj)` it is important to not iterate over the properties and accidentally get the prototype propeties. Use `hasOwnProperty` to safeguard against this.

```javascript
for (var prop in obj) {
  if (obj.hasOwnProperty(prop)) {
    // do stuff with prop
  }
}
```

Could potentially avoid `hasOwnProperty` collision by taking it from the object prototype and caching it:

```javascript
var hasOwn = Object.prototype.hasOwnProperty;
```

# Chapter 3: Literals and Constructors

## Objects

Can directly set the properties on an object, override them later, and delete the properties...

```javascript
var dog = {
  bark: false,
  say: function() {
    alert('woof!');
  }
};
dog.bark = true;
delete dog.bark;
```

## Custom constructor functions

Constructors can be made for custom objects, creating a `Person` constructor looks like this:

```javascript
var Person = function(name) {
  this.name = name;
};
```

- The `prototype` of the object is inherited
- Properties/methods are added to the `this` property in the constructor
- Object is implicitly returned (and any object can be returned, not bound to `this`)

When creating a method that will be on all instances of the object, use the `prototype`:

```javascript
Person.prototype.say = function() {
  return "I am " + this.name;
}
```

Stop callers from accidentally forgetting to call new, `this` will point to the global context...

- Could use `that` object and return that instead

```javascript
function Waffle() {
  var that = {};
  that.name = "waffle";
  return that;
}
```

- Could use *self-invoking constructor* that will detect whether `new` is being used

```javascript
function Waffle() {
  if (!(this instanceof Waffle)) {
    return new Waffle()
  };
  // else do the normal thing
}
```

## Array literals

Don't use `new Array()` as the first argument will become the length of the array with no elements

Arrays are still `Object`s, to assert that the object is indeed an array we will have to use one of the following:

- ES5 adds support for Array.isArray()
- Use `Object.prototype.toString` should return `[object Array]`
- **Don't use** length/slice or method detection, other objects can implement this

## Regular expression literals

There are two ways to create a regular expression literal

- `new Regexp()` constructor
- Regular expression literal (`/foo/g`)

Stick to the literal notation since quotes need to be escaped in the constructor version

## Primitive wrappers

Three primitive values have wrapper objects (`Number`, `String`, and `Boolean`) and are not primitive values.

```javascript
var n = new Number(100);
typeof n === "object";

var n = 100;
typeof n === "number";
```

## Error objects

Number of build-in errors: `Error()`, `SyntaxError()`, `TypeError()`.

There are properties `name` and `message`, however `throw` will accept any object

# Chapter 4: Functions

Functions can be...

- Dynamically created at runtime
- Assigned to variables
- Passed as arguments
- Have their own properties and methods

Functions are really just objects, take a look at `new Function(<str>)`, however it is a bad practice. This is basically equivalent to executing `eval`.

Functions can also be named expressions...

```javascript
function foo() {}; // declaration
var bar = function() {}; // anonymous
var baz = function bar() {}; // named

foo.name === "foo";
bar.name === "";
baz.name === "baz";
```

Function expressions only hoist the variable, can leave the user a bit confused...

```javascript
function bar() {};

function hoisted() {
  console.log(bar); // undefined
  var bar = function() {}; // doh! the *declaration* is only hoisted
}
```

## Callback pattern

Functions are objects and can be dynamically passed as an argument.

```javascript
function foo(cbk) {
  if (typeof cbk === "function") {
    cbk();
  }
}

foo(function() { boom(); });

```

Problem with callbacks is that if they depend on an existing scope

- Could pass this as a parameter `foo('bar', context)`
  - This means the call site must apply the context, `callback.call(context)`

Callbacks are popular in event loops (listeners on the DOM) and timeouts

- Event listeners: `document.addEventListener("click", console.log)`
- Timeouts: `setTimeout(callback, 500)`


## Returning and self-defining functions

Functions can also be returned from functions, the returned function could close over a scope or do a one-off initialization.

```javascript
function setUp() {
  doThisOnce();

  setUp = function() {
    logAlreadyDone();
  }
}
```
applicationWillRedesignActive
Tricky thing is that self-defining functions can be re-assigned and have no effect.

```javascript
var setupForever = setup;

setupForever(); // do this once called
setupForever(); // do this once called again
```

## Immediate functions

When a function is immediately invoked after it is declared, it is known as an *immediate function*

```javascript
(function() {
  console.log("WHOA!");
}());
```

- Wrapping the function in parentheses can let the reader know that it will be immediately called
- The self-invocation means that the *scope* is sandboxed to the function (no global leakage)
- Scopes can be passed as arguments to increase interoperability between environments

```javascript
(function (global) {
  // the global object is now referred by `global` in this scope
}(this));
```

## Immediate object initialization

This pattern creates an object and immediately calls a method on it...

```javascript
({
  init: function() {
    // do some init stuff
  }
}).init();
```

This pops out the initialized state without having to worry about global state pollution. Both this and immediate functions are to help with accidentally throwing into the global scope.

## Init time branching

Sniff for properties or "do-it-once" activities during initialization

```javascript
// Every time utils.corgi is needed we have already computed what it needs to be
utils = { };

if (typeof window.corgi === "object") {
  utils.corgi = function() {};
} else {
  utils.corgi = false;
}
```

## Memoization for functions

Save computations so that they can be used for later

```javascript
var functor = function() {
  if (functor.corgi) {
    return functor.corgi;
  }
  functor.corgi = computeThousandsOfCorgis();
  return functor.corgi;
}
```

## Configuration objects

Take multiple arguments and use a configuration object instead

Pros

- No need to maintain argument order
- Easier to maintain
- Optional parameters

Cons

- You need to know the names of the parameters
- Property names cannot be minified

## Partial function application and currying

Function application means that we can *apply* a function to a context with arguments

```javascript
// similar to basically doing...
pseudoArray.slice();

// except if the function is not available, we can "apply" an implementation from the Array prototype
Array.prototype.slice.apply(pseudoArray);
```

- apply: array
- call: no array

# Chapter 5: Object Creation Patterns

## Namespace pattern

Reduce the number of globals by using this pattern where we namespace on a declared global object...

```javascript
var APP = {};
APP.foo = function() {};
APP.bar = function() {};

// better than...
foo = function() {};
bar = function() {};
```

## General purpose namespace function

Useful to have a function that creates the namespaces for us when nesting is deep...

```javascript
APP.namespace("APP.foo.bar");

// semantically the same as something that creates
APP.foo = {
  bar: {
  }
};
```

A sample implementation:

```javascript
APP.namespace = function(ns_string) {
  var parts = ns_string.split('.'),
      parent = APP,
      i;

  if (parts[0] === "APP") {
    parts = parts.slice(1);
  }

  for (i = 0; i < parts.length; i++) {
    if (typeof parents[parts[i]] === "undefined") {
      parent[parts[i]] = {};
      parent = parent[parts[i]];
    }
  }

  return parent;
};
```

## Declaring dependencies

Declare dependencies that the function/module requires at the top

- Explicit declaration tells future users what is used
- Makes it easy to find/resolve dependencies
- Local var means no resolution lookup
- Minification tools can rename local vars

```javascript
function bar() {
  var event = lib.Event,
      dom = lib.DOM;
}
```

## Private properties and methods

No real sense of *privacy* in JS, can be imitated with a closure instead

```javascript
function Foo() {
  var bar = "secret";

  this.getBar = function() {
    return bar;
  }
}
```

Gotchas:

- Environments such as Rhino/FF can use second argument to `eval` to sneak into context
- If passing an object/array from a privileged `get*` method, pass by reference allows mutation

Also possible with object literals when used with immediate functions...

```javascript
var obj;
(function() {
  var secret = "bar";

  obj = {
    getSecret: function() {
      return secret;
    }
  }
}());
```

### Prototypes and privacy

Problem when adding to constructor since it's recreated every time

Putting the private members used on an immediate function for the `prototype` will simulate this behavior:

```javascript
Gadget.prototype = (function() {
  var browser = "Chromez"; // private to this scope

  return {
    getBrowser: function() {
      return browser;
    }
  }
});
```

### Revealing private functions in public API

Put the functions you want in an immediate function and expose what you want on the object at the end...

```javascript
var api;

(function() {
  function doh() {}
  function bar() {}
  // choose here what we want to reveal
  api = {
    foo: doh,
    bar: bar
  }
});
```

## Module pattern

Composes several patterns:

- Namespaces
- Immediate functions
- Private/privileged members
- Declaring dependencies

Use the immediate function to maintain scope and return the module object interface

```javascript
APP.dom = (function() {
  var privateMember = "foo";

  return {
    getPrivateMember: function() {
      return privateMember;
    }
  }
}());
```

### Revealing module pattern

Choose from functions inside of the immediate functions which to expose in the API, this "reveals" the API that the user chooses

```javascript
APP.dom = (function() {
  var privateMember = "foo";

  function getPrivateMember() {
    return privateMember;
  }

  return {
    getPrivateMember: getPrivateMember
  }
}());
```

### Modules that create constructors

Instead of returning an object we can return a function and handle private state within the immediate function

```javascript
APP.dom = (function() {
  var privateMember = "foo";

  var dom = function() {};

  dom.prototype = {
    version: "1.2"
  };

  return dom;
}());
```


## Sandbox pattern

Addresses two pain points of module pattern:

- Relying on single global variable, no way to have two versions of single variable on the page
- Nested variables resolved constantly at runtime (`APP.dom.Event.bindEvent`)

### Global constructor

The single global object is a constructor, objects are created within a callback passed in

```javascript
new Sandbox(function(box) {
  // sandboxed code
});
```

Could omit `new` with enforcing new and have module names passed in as dependencies for code inside box

```javascript
Sandbox(['ajax', 'event'], function(box) {});
```

### Adding modules

Modules could be added to the Sandbox since it is also just a function object

```javascript
Sandbox.modules = {};

Sandbox.modules.dom = function(box) {
  // dom stuff;
}
```

### Sandbox implementation

```javascript
function Sandbox() {
  var args = Array.prototype.slice.call(arguments),
      calllback = args.pop(),
      modules = (args[0] && typeof args[0] === "string") ? args : args[0],
      i;

    if (!(this instanceof Sandbox)) {
      return new Sandbox(modules, callback);
    }

    // add properties to object here as needed
    this.foo = "bar";

    // add modules to the object
    // default/* to all modules
    if (!modules || modules === "*") {
      modules = [];
      for (i in Sandbox.modules) {
        if (Sandbox.modules.hasOwnProperty(i)) {
          modules.push(i);
        }
      }
    }

    for (i = 0; i < modules.length; i++) {
      Sandbox.modules[modules[i]](this);
    }

    callback(this);
}
```

## Static members

### Public static members

Simply add these to the constructor object

```javascript
var App = {};
App.start = function() {}; // static
```

### Private static members

Add the members in the constructor and return a new constructor closed over them

```javascript
var App = (function() {
  var counter = 0,
      newApp;

  newApp = function() {
    counter += 1;
  }

  return newApp
}());
```

## Chaining pattern

Returning `this` will allow methods to be chained

- Pro: less typing
- Pro: leads to smaller more specialized functions
- Con: debugging a specific line of code is pretty much impossible

# Chapter 6: code reuse patterns

## Classical vs prototypal inheritance patterns

Javascript supports prototypal, people bastardize it and turn it into classical

## Classical patterns

**Default pattern**

```javascript
function inherit(C, P) {
  C.prototype = new P();
}
```

Follows prototype chain up when members or function are called.

Drawback

- Inherits everything including properties to instance all the way up the chain

**Rent-a-Constructor**

Defines a constructor using it's parents constructors

```javascript
function CatBird() {
  // could forward arguments as well...
  Cat.apply(this);
  Bird.apply(this);
}
```

Pros/cons

- Con: nothing gets inherited from prototype
- Pro: get true copies with no other members

**Rent and Set prototype**

Take the constructor and set the child prototype to a new instance of the constructor

```javascript
function Child() {
  Parent.apply(this, arguments);
}

Child.prototype = new Parent();
```

Drawback is that the parent's prototype gets called twice

**Share the prototype**

Idea is that anything worth inheriting should be in the prototype

```javascript
function inherit(C, P) {
  C.prototype = P.prototype;
}
```

Drawback: children are affected downstream if the prototype of the parent is touched

**A temporary constructor**

Inherit what you want from the constructor at that time by assigning to a vanilla object

```javascript
function inherit(C, P) {
  var F = function() {};
  F.prototype = P.prototype;
  C.prototype = new F();
}
```

## Prototypal inheritance

Here objects inherit from other objects, reusing the objects should be creating a new copy of them and taking the prototype

```javascript
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
```

ES5 adds `Object.create` to the language, you don't need to roll your own anymore!

### Inheritance by copying properties

Simply copies properties from one object to another

```javascript
function extend(parent, child) {
  var i;
  child = child || {};

  for (i in parent) {
    if (parent.hasOwnProperty(i)) {
      child[i] = parent[i];
    }
  }
  return child;
}
```

Above is a shallow copy, if it hits an Array or object it simply copies the reference. Below is an implementation of deepExtend

```javascript
function deepExtend(parent, child) {
  var i,
      toStr = Object.prototype.toString,
      astr = "[object Array]";

  child = child || {};

  for (i in parent) {
    if (parent.hasOwnProperty(i)) {
      if (typeof parent[i] === "object") {
        child[i] = (toStr.call(parent[i]) === astr) ? [] : {};
        extendDeep(parent[i], child[i]);
      } else {
        child[i] = parent[i];
      }
    }
  }

  return child;
}
```

### Mixin

Taking the idea of property copying a bit further and infusing objects from one into another

```javascript
function mix() {
  var arg, prop, child = {};

  for (arg = 0; arg < arguments.length; arg += 1) {
    for (prop in arguments[arg]) {
      if (arguments[arg].hasOwnProperty(prop)) {
        child[prop] = arguments[arg][prop];
      }
    }
  }
}
```

## Borrowing methods

Sometimes necessary to take a method or two, don't need a relationship

Can use `call` and `apply`

For literal support, could literal (`[].slice.call()`) or refer to the prototype directly (`Array.prototype.slice.call`)

### Binding methods

Since methods could be called without a context directly (`say()`), we can bind an object always to a method invocation

```javascript
function bind(o, m) {
  return function() {
    return m.apply(o, [].slice.call(arguments));
  }
}
```

### Functional binding

ES5 provides `Function.prototype.bind` which allows for context to be added to a function.

Shim:

```javascript
Function.prototype.bind = function(thisArg) {
  var fn = this,
      slice = Array.prototype.slice,
      args = slice.call(arguments, 1);
  return function() {
    return fn.apply(thisArg, args.concat(slice.call(arguments)));
  }
}
```

# Chapter 7: Design Patterns

## Singleton

Only have one instance of a class, another instantiation should return the same object

Put a property on the constructor:

```javascript
function Universe() {
  if (typeof Universe.instance === "object") {
    return Universe.instance;
  }

  Universe.instance = this;
}
```

Can also return the instance in a closure that overwrites the constructor:

```javascript
function Universe() {
  var instance = this;

  Universe = function() {
    return instance;
  }
}
```

Drawbacks

- `constructor` property changes
- Between instantations the properties are lost

Could assign the prototype to the `this` in the constructor and reset the constructor pointer

```javascript
function Universe() {
  var instance;

  Universe = function Universe() {
    return instance;
  }

  Universe.prototype = this;

  instance = new Universe();
  instance.constructor = Universe;

  return instance;
}
```

Could also wrap in IIFE:

```javascript
(function() {
  var instance;

  Universe = function() {
    if (instance) {
      return instance;
    }

    instance = this;
  }
}());
```

## Factory

Creates objects in a similar way:

- Performs repeated operations
- Customers of factories can get an object without explicitly knowing the type

Example where we have:

- Common CarMaker constructor
- Static method `factory` to create car objects
- Specializations of car (Compact, SUV, etc)

Should be able to pass the type into the factory method and get that object back out

```javascript
var corolla = CarMaker.factory('compact');
var solstice = CarMaker.factory('convertible');
```

Implementation

```javascript
function CarMaker() {}

CarMaker.prototype.drive = function() {
  return this.doors;
}

CarMaker.factory = function(type) {
  var constr = type,
      newcar;

      if (typeof CarMaker[constr].prototype.drive !== 'function') {
        CarMaker[constr].prototype = new CarMaker();
      }

      newcar = new CarMaker[constr]();

      return newcar;
}

CarMaker.Compact = function() {
  this.doors = 4;
}
```

Also provide `new Object` as an "in the wild" factory that passes primitives and creates the wrapper around it

## Iterator

Provide iteration through a `next` method that returns the next element in the sequence

```javascript
var el;
while (el = agg.next()) {
  // do stuff
}
```

Could provide the null check through `hasNext` method

```javascript
var agg = (function() {
  var index = 0,
      data = [...],
      length = data.length;

  return {
    next: function() {
      if (!this.hasNext()) {
        return null;
      }

      element = data[index];
      index += 2;
      return element;
    },

    hasNext: function() {
      return index < length;
    }
  }
});
```

## Decorator

Additional functionality added at runtime, usually a challenge with static classes.

Want to add the functionality as you go along, start with an object and *decorate* it with what you want

```javascript
var sale = new Sale(100);
sale = sale.decorate('fedtax');
sale = sale.decorate('cdn');
sale.getPrice();
```

### Implementation

Could have each decorator have methods that are overridde, inherit the previous object and call that before you do your thing

```javascript
Sale.prototype.getPrice = function() {
  return this.price;
};

Sale.decorators.fedtax = function() {
  var price = this.uber.getPrice();
  price += price * 0.05;
  return price;
};
```

This means that decorate will tie together what comes in and the associated decoration method

```javascript
Sale.prototype.decorate = function(decorator) {
  var F = function() {},
      overrides = this.constructor.decorators[decorator],
      i, newobj;
  F.prototype = this;
  newobj = new F();
  newobj.uber = F.prototype;
  for (i in overrides) {
    if (overrides.hasOwnProperty(i)) {
      newobj[i] = overrides[i];
    }
  }
}
```

### Implementation with a list

Let's implement without using inheritance

```javascript
function Sale(price) {
  this.price = 100;
  this.decorators_list = [];
}

Sale.decorators = {};

Sale.decorators.fedtax = {
  getPrice: function() {...}
}

Sale.prototype.decorate = function (decorator) {
  this.decorators_list.push(decorator);
}

Sale.prototype.getPrice = function() {
  var price = this.price,
      i,
      max = this.decorators_list.length,
      name;

    for (i = 0; i < max; i++) {
      name = this.decorators_list[i];
      price = Sale.decorators[name].getPrice(price);
    }
}
```

## Strategy

Select algorithms or "strategies" at runtime, allows for dependency on context to be removed

## Facade

Simply putting a collection or single methods behind another interface

```javascript
// We always do this when we need to stop the browser from doing something...
e.preventDefault();
e.stopPropagation();

// We could also wrap it behind a "stop" method
function stop() {
  e.preventDefault();
  e.stopPropagation();
  // handle other cross-browser weirdness
}
```

## Proxy

One object acts as an interface to another one, sits as a client to an object to protect access

Proxy example: *lazy initialization*

Could put the proxy inbetween client and actual object, proxy calls for initialization to do nothing, when an action happens *then* initialize the object (may never even have to do this!)

Proxy is also an example of a cache, can remember previous values that the client uses and off-loads the server/object by returning remembered value

## Mediator

Don't let objects talk to each other (this is *coupling*), let them talk to the mediator who knows how to hand off messages

```javascript
Player.prototype.play = function() {
  this.points += 1;
  mediator.played();
}

mediator = {
  played: function() {
    scoreboard.update(score);
    // do other mediator things
  }
}
```

## Observer

Also known as pub/sub.

Promotes loose coupling and publisher notifies all subscribers/observers of events that happen

```javascript
var publisher = {
  subscribers: {
    any: []
  },

  on: function(message) {
    for (var subscriber in subscribers) {
      subscriber.notify(message);
    }
  },

  subscribe: function(obj) {
    subscribers.push(obj);
  }
}
```

# DOM and browser patterns

## Separation of concerns:

- *Content*: HTML
- *Presentation*: CSS
- *Behavior*: JS

These separation of concerns vibe with *progressive enhancement* where we start with the most basic experience, and as the compabilities of the user increases we can add support for additional features (CSS) and if it supports JS we can turn that on

Javascript should be *unobstrusive*, the page should not be unusable in browsers that do not support it

Compability detection means not using UA sniffing but checking for the existence of some feature (`document.attachEvent` vs `navigator.userAgent.indexOf('MSIE')`)

## DOM scripting

Keep DOM access to a minimum

- Avoid it in loops
- Assign to local variables
- Use selectors API when available
- Cache length when using HTML collections!

Repaints and reflows are expensive, keep them to a minimum by creating fragments and appending children to them later

## Events

Event delegation: hit an outer element and delegate to who should actuall get it, avoids binding and removing elements (think of binding to a `ul` and delegating to `li`s instead of binding each one)

## Long-running scripts

Simulate scripts that take a long time by either using setTimeout or web workers (there are no threads)

- setTimeout: break into smaller chunks and keep the UI responsive
- web workers: offload into another thread and use `onmessage` and `postMessage` from the worker to communicate

## Remote scripting

### XHR

Set up `XMLHttpRequest`, provide a callback, and send the request

```javascript
var xhr = new XMLHttpRequest();

xhr.onreadystatechange = handleResponse;
xhr.open("GET", "page.html", true);
xhr.send();
```

### JSONP

JSON with padding also allows for another way to make remote requests, not restricted by same-domain policy

JSONP would look something like `http:///www.foo.org/getdata.php?callback=foo`

This means that the `script` element will be created and appended to the DOM

```javascript
var script = document.createElement('script');
script.src = url;
document.body.appendChild(script);
```

The payload will be JSON data wrapped in the parameterized callback (`foo({hey: 'there'})`)

