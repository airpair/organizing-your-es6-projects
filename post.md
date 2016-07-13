##### Introduction
ES6, the latest incarnation of Javascript, isn't fully baked into browsers yet.  As a developer who loves using the Chrome debugger, it can't come fast enough.  What follows is a quick reference to common patterns and problems working with the current best practices for ES6.

##### Babel
In many ways, the real heroes of the Javascript world have been the transpilers.  And the best of breed is Babel.  This open source project has plugins for every variation of Javascript, Typescript, Coffeescript, Emberscript, JSX, ES6 and much more.  The goal is the same with each.  Provide a backwards compatible way to convert all these new syntaxes into ES5, which all browsers support.  ES5 can do (almost) everything that these cleaner new syntaxes can do, using more code.  Babel provides a cross browser way to get the future, while still having to support the present.  

##### Webpack
An equally important upgrade from the Facebook brain trust that's working on React & Babel.  Webpack is the asset pipeline, borrowing heavily from the legacy of Browserify and RequireJS.  Its many paradigm shifting features are beyond the scope of this article, but for our current purposes, Webpack is the framework that runs Babel to transpile javascript from ES6 down to ES5. and manages loading dependencies for front end (js, css, fonts) assets.  

##### import, require and export
Javascript 6 eschews the idea of a global scope for classes to share by default.  Unlike ruby or Java, ES6 uses a module system of `import` and `export` to setup it's dependencies.

```javascript
import React from 'react'

export default ComponentName extends React.Component
```

This is simple as it gets.  There are only a few things to notice about this.  First, `'react'` points to a file in the project's local `npm-modules` directory.  If you're looking to import something from the current directory the syntax is:

```javascript
import something from './filename'
```

Notice you don't need the `.js` or `.jsx` in ES6.  It assumes it's javascript.

##### Importing other file types (fonts, images)

Everything else can be imported as well using the `require` function, which emulates the CommonJS pattern for other file types.  This is a Webpack feature, not specific to ES6.  But under the covers, Babel turns all `imports` into `requires` anyway.

```javascript
require('./style-file.css');
require('../fonts/font-file.ttf');
```

Notice the relative path to the font's directory here.  If the path to the required file is too far away, it's a great opportunity to create an npm package out of it and include it more simply, as we do with react or any other npm package. 

##### Basic Syntax of Classes

Javascript makes classes a fairly simple extention of the usual object.  For example, this is an object in javascript.

```javascript
var Hello = {};
```

And this is a class.

```javascript
var Hello = class {};
```

Whoa.  So what's the difference.  Well, OO languages like Ruby, there's an `initialize` method, that sets default properties on instantiation.   Javascript has one of these as well.

```javascript
var Hello = class {

  constructor(message) {
    this.message = message;
  }
};
```

The constructor will set the message property to whatever we set it to upon creating a new message.

```javascript
var greeting = new Hello("Bonjour");
```

There's also now the ability to extend (or inherit) from a parent class.  In react we `extend` `React.Component` when we create a new component.  That's done with `super`.

```javascript
class MyComponent extends React.Component {

  constructor(props) {
    super(props)
  }
}
```

Here we're calling the React.Component constructor method and passing in the child's props.  This is where you'll see `super` used most.  But if we wanted to call a parent's method within any other function, we could do that with `super` within that function as well.

##### Library classes for your components

When building top-level components, (sometimes called containers) it's very easy to end up with a very long class that handles a lot of the application logic.  As I'm sure you can guess, scrolling around looking for this method or that one can get tedious.  So let's use the power of import and export to organize our Javascript.

First thing to realize is that export doesn't care what file it's in or how many other exports are in that file. Likewise, import only cares about the path to a file.  The one caviat is, if you have multiple exports in a single file, import needs them to be named.

```javascript
// Utility library
export function color() {}
export function shape() {}
```

```javascript
// Component
import { color, shape } from './utilities'

color()
shape()
```

`color` and `shape` are yanked out of that utility-library file one by one and placed into the component's file.

We can import everything using an asterisk.

```javascript

import * as util from './utilities'

util.color()
util.shape()
```

As?  Yes `as`.  This handy command allows us to rename as we import.

```javascript

import React as Tcaer from 'react'

class MyComponent extends Tcaer {
}
```

Not terribly readable for your fellow developers in this example, but it's a freedom you can use.

The last example is encapsulating any of these utility functions in a helper class.  We use the static for this.

```javascript
// utilities.js
export default class {
  static color() {}
  static shape() {}
}

// Component
import Utilities from './utilities'

Utilities.color()
Utilities.shape()
```

Static methods are called on the class itself.  They cannot be called if one were to create an instance of Utilities.

```javascript
// Doesn't work

var myUtil = new Utilities()
myUtil.color();  // error
```

So using `static` provides a nice namespacing for groups of methods that share common functionality. 

Finally, we can also import using the dot notation.

```javascript
// Component
const color = require('./utilities').color;
color()
```

#### Conclusion
As we work with ES6 and React, we're going to want to stuff all those methods that don't pertain to the UI into helper library files.  These methods can be tested independently of React and user interaction.  They should accept arguments and return something React can use.  Here's a final more full-throated example to put everything in context.

```javascript
// cruncher.js
export default class {
   static addUp(a, b) {
     return a + b
   }
}

// Component
import React from 'react'
import NumberCruncher from './cruncher'

export default class Calculator extends React.Component {
    constructor() {
      super(props)
      this.calculate = this.calculate.bind(this)
    }

    calculate() {
      const a = this.refs("inputA")
      const b = this.refs("inputB")
      return NumberCruncher.addUp(a, b)
    }

    render() {
      <div className="calc">
         <input type="text" ref="inputA" />
         <input type="text" ref="inputB" />
         <button onClick={this.calculate} />
      </div>
    }
}
```

#### Further Reading
* http://www.2ality.com/2014/09/es6-modules-final.html