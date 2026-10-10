## Two ways to import a node module

- **Common JS** - this is the default and the original way to import modules
in a node program
    - uses the ```require()``` to import modules
    - uses ```module.exports``` property to export module objects/variables
    - it can also use the ```exports``` (note the 's') to export module objects/variables
    - what nodejs uses by default

- ES6 JS (ES modules / ESM) - this was added (without an experimental flag) in node.js v13.2 / v12.17 to support the ES6 syntax way of
importing ES6 modules
    - uses the ```import``` to import modules
    - uses the ```export``` to export module objects/variables
    - can be used in files that have ```.mjs``` as their extension, or in `.js` files when the nearest `package.json` has `"type": "module"` (see below)

The two module systems are not fully compatible, so this is somewhat tricky to do.

Node needs to know—before it loads a module—whether that
module will be using ```require()``` and ```module.exports``` or if it
will be using ```import``` and ```export```.

-----

## Caveat
When Node loads a file of JavaScript code as a CommonJS module, it automatically defines the
require() function along with identifiers exports and module,
and it does not enable the import and export keywords. On the
other hand, when Node loads a file of code as an ES6 module, it must
enable the import and export declarations, and it must not define
extra identifiers like require, module, and exports.

-----

## using package.json to enable either CommonJS or ES6JS way of modularity

For files that _do not have an explicit .mjs or .cjs extension_, Node looks for a file named **package.json** in the same directory as the file and then in each of the containing directories.

Once the nearest **package.json** file is found, Node checks for a top-level type ***property*** in the **JSON object**.

- If the value of the ***type property*** is **“module”**, then Node loads
the file as an **ES6 module**.

- If the value of that ***type property*** is **“commonjs”**, then Node loads the file as a **CommonJS module**.

Note that you do not need to have a package.json file to run Node programs:

- when **no such file is found** (or when the file is **found** but it does **not have** a type property), Node **defaults** to using **CommonJS modules**.

This package.json trick only becomes necessary if you want to use ES6 modules with Node and do not want to use the .mjs file extension.

-----

## ES6 module can load CommonJS module, but not (easily) the other way around

Because there is an enormous amount of existing Node code written
using CommonJS module format, Node allows ES6 modules to load
CommonJS modules using the import keyword. The reverse was not
true for a long time, however: a CommonJS module could not use require() to load
an ES6 module, it had to use the asynchronous dynamic `import()` function instead.

**Update:** newer Node.js versions (v22.12+, and v20.19+) **can** `require()` an ES module,
as long as that module (and its dependencies) doesn't use top-level `await`.
