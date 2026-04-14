---
name: js
description: >
  JavaScript development skill. Produces clean, modern, secure, and maintainable
  JavaScript following the MDN Writing Style Guide and MDN JavaScript Code Style
  Guide. Enforces `const`/`let` over `var`, modern syntax, strict equality,
  proper error handling, and secure patterns.
  TRIGGER when: project contains *.js/*.mjs/*.cjs files, package.json, or the
  user asks to write, review, or fix JavaScript code.
  DO NOT TRIGGER when: the project is TypeScript-only (use node-ts), or the
  task is about a non-JS language.
---

# JavaScript Writing Best Practices

> Based on [MDN Writing Style Guide](https://developer.mozilla.org/en-US/docs/MDN/Writing_guidelines/Writing_style_guide) and [MDN JavaScript Code Style Guide](https://developer.mozilla.org/en-US/docs/MDN/Writing_guidelines/Writing_style_guide/Code_style_guide/JavaScript)

When writing JavaScript code, follow these guidelines to produce clean, modern, secure, and maintainable code.

---

## Variable Declarations

### Use `const` and `let`, never `var`

- **`const`** — default choice; use for variables that won't be reassigned
- **`let`** — use only when reassignment is needed
- **`var`** — never use; it leaks into function/global scope and causes hoisting bugs

```javascript
// GOOD
const name = "Shilpa";
let age = 40;
age++;

// BAD
var age = 40;
let name = "Shilpa"; // should be const since it's never reassigned
```

### Declare one variable per line

```javascript
// GOOD
let var1;
let var2;
let var3 = "Apapou";

// BAD — chained assignment leaks globals in non-strict mode
let var3 = var4 = "Apapou";
```

### Declare variables at point of first use

Don't hoist declarations to the top of a function. Declare close to where the variable is used, within the narrowest possible scope.

---

## Naming Conventions

### Variables — `camelCase`, lowercase start

- Use semantic, human-readable names (3–10 characters when possible)
- No Hungarian notation (`bBought`, `sName`, `nCount`)
- Plurals for collections, no type suffixes
- No articles or possessives

```javascript
// GOOD
const playerScore = 0;
const speed = distance / time;
const cars = ["Tesla", "BMW"];

// BAD
const carArray = ["Tesla", "BMW"];
const myCar = "Tesla";
const acclmtr = 100;
```

### Functions — `camelCase`, lowercase start

Use concise, verb-based names that describe intent.

```javascript
// GOOD
function sayHello() { }
function calculateTotal() { }

// BAD
function SayHello() { }    // PascalCase is for classes
function doIt() { }         // vague
```

### Classes — `PascalCase`

```javascript
class Person { }
class HttpClient { }
```

### Constants (true constants) — `UPPER_SNAKE_CASE`

```javascript
const MAX_USERS = 100;
const API_BASE_URL = "/api/v1";
```

---

## Functions

### Prefer function declarations over function expressions

```javascript
// GOOD
function sum(a, b) {
  return a + b;
}

// BAD
let sum = function (a, b) {
  return a + b;
};
```

### Use arrow functions only as anonymous callbacks

```javascript
// GOOD — arrow as callback
const doubled = array.map((n) => n * 2);
const sum = array.reduce((a, b) => a + b);

// GOOD — named function via declaration
function processData(data) {
  // ...
}

// BAD — arrow assigned to identifier
const processData = (data) => {
  // ...
};
```

### Use implicit return in arrow functions when possible

```javascript
// GOOD
arr.map((e) => e.id);

// BAD — unnecessary block body
arr.map((e) => {
  return e.id;
});
```

### Comment out unused parameters

```javascript
array.forEach((value /* , index, array */) => {
  // ...
});
```

---

## Strings

### Use template literals for interpolation

```javascript
// GOOD
console.log(`Hi! I'm ${name}!`);

// BAD
console.log("Hi! I'm " + name + "!");
```

### Use plain string literals when there's no substitution

```javascript
// GOOD
const message = "Hello world";

// UNNECESSARY
const message = `Hello world`;
```

### Use template literals for multiline strings

```javascript
const text = `This spans
multiple lines
without escape characters`;
```

---

## Arrays

### Use literals, not constructors

```javascript
// GOOD
const items = [];

// BAD
const items = new Array();
```

### Use `push()` to add items

```javascript
// GOOD
pets.push("cat");

// BAD
pets[pets.length] = "cat";
```

### Prefer semantic array methods over loops

Use `map()`, `filter()`, `find()`, `findIndex()`, `every()`, `some()`, `includes()`, `reduce()` when they express intent more clearly than a loop.

---

## Objects

### Use literals, not constructors

```javascript
// GOOD
const obj = {};

// BAD
const obj = new Object();
```

### Use ES class syntax, not prototype-based constructors

```javascript
// GOOD
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greeting() {
    console.log(`Hi! I'm ${this.name}`);
  }
}

// BAD
function Person(name, age) {
  this.name = name;
  this.age = age;
}
Person.prototype.greeting = function () { };
```

### Use method shorthand

```javascript
// GOOD
const obj = {
  foo() { },
  bar() { },
};

// BAD
const obj = {
  foo: function () { },
  bar: function () { },
};
```

### Use property shorthand

```javascript
// GOOD
function createObject(name, age) {
  return { name, age };
}

// BAD
function createObject(name, age) {
  return { name: name, age: age };
}
```

### Use `Object.hasOwn()` instead of `hasOwnProperty()`

```javascript
// GOOD
Object.hasOwn(obj, "property");

// DEPRECATED
obj.hasOwnProperty("property");
```

---

## Loops and Iteration

### Prefer `for...of` or `forEach()` over classical `for`

```javascript
// GOOD
for (const dog of dogs) {
  console.log(dog);
}

dogs.forEach((dog) => {
  console.log(dog);
});

// BAD
for (let i = 0; i < dogs.length; i++) {
  console.log(dogs[i]);
}
```

### Always declare loop variables

```javascript
// GOOD
for (const cat of cats) { }
for (let i = 0; i < 4; i++) { }

// BAD — creates implicit global
for (i of cats) { }
```

### Never use `for...in` on arrays or strings

`for...in` iterates over enumerable properties, not indices — it includes inherited properties and may iterate in unexpected order.

---

## Conditionals

### Use ternary for simple value assignments

```javascript
// GOOD
const x = condition ? 1 : 2;

// BAD
let x;
if (condition) {
  x = 1;
} else {
  x = 2;
}
```

### Omit `else` after `return`

```javascript
// GOOD
if (test) {
  return value;
}
// Remaining code handles the false case

// BAD
if (test) {
  return value;
} else {
  // ...
}
```

### Always use braces with control flow

```javascript
// GOOD
for (const car of cars) {
  car.paint("red");
}

// BAD
for (const car of cars) car.paint("red");
```

### Use shortcuts for boolean tests

```javascript
// GOOD
if (x) { }
if (!x) { }

// BAD
if (x === true) { }
if (x === false) { }
```

---

## Switch Statements

### Return instead of break when possible

```javascript
switch (species) {
  case "chicken":
    return farm.shed;
  case "horse":
    return corral.entry;
  default:
    return "";
}
```

### Scope variables in case blocks with braces

```javascript
switch (fruit) {
  case "Orange": {
    const slice = fruit.slice();
    eat(slice);
    break;
  }
  case "Apple": {
    const core = fruit.extractCore();
    recycle(core);
    break;
  }
}
```

---

## Operators and Equality

### Use strict equality (`===`, `!==`)

```javascript
// GOOD
name === "Shilpa";
age !== 25;

// BAD
name == "Shilpa";
age != 25;
```

**Exception:** `== null` is acceptable to check for both `null` and `undefined`. Add a comment explaining why.

### Use explicit type conversion

```javascript
// GOOD
this.name = String(name);
this.year = Number(year);

// BAD — implicit coercion
this.name = "" + name;
this.year = +year;
```

### Always include radix in `parseInt()`

```javascript
parseInt("101", 10);  // 101
parseInt("101", 2);   // 5
```

---

## Async Code

### Prefer `async`/`await` over raw Promises

```javascript
// GOOD
async function getData() {
  try {
    const response = await fetch("/api/data");
    return response.json();
  } catch (error) {
    console.error(error);
  }
}

// BAD — promise chain
function getData() {
  return fetch("/api/data")
    .then((response) => response.json())
    .catch((error) => console.error(error));
}
```

### Avoid top-level `await` unless in an ES module context

Wrap in an async function for maximum compatibility.

---

## Error Handling

### Use `try...catch` for recoverable errors

```javascript
try {
  const result = getResult();
  console.log(result);
} catch (error) {
  console.error(error);
}
```

### Omit the `catch` parameter if you don't need it

```javascript
try {
  console.log(getResult());
} catch {
  console.error("An error happened!");
}
```

### Let non-recoverable errors bubble up

Only catch errors you can meaningfully handle. Don't swallow exceptions silently.

---

## Comments

### Use single-line comments (`//`), not block comments (`/* */`)

```javascript
// GOOD — single-line comment
// Calculate the sum of the first four elements

/* BAD — block comment for regular comments */
```

**Exception:** Use `/* */` only for commenting out unused function parameters.

### Comment guidelines

- Add comments when purpose or logic isn't obvious from the code
- Don't restate what the code already says
- Start with a capital letter, no trailing period
- Place comments on a separate line above the code they refer to

```javascript
// GOOD — explains why
// Skip the first element because it's the header row
for (let i = 1; i < rows.length; i++) { }

// BAD — restates the code
// Loop from 1 to length
for (let i = 1; i < rows.length; i++) { }
```

### Comment log output after the call

```javascript
console.log(fruitBasket); // ['banana', 'mango', 'orange']
```

### Use `// ...` for omitted code

```javascript
function exampleFunc() {
  // Add your code here
  // ...
}
```

### Multi-line comments: use `//` on each line

```javascript
// This function has some unusual limitations
// that are worth calling out:
// Limitation 1: ...
// Limitation 2: ...
```

---

## DOM and Web APIs

### Use `textContent`, not `innerHTML`, for text

```javascript
// GOOD — safe
para.textContent = text;

// BAD — XSS risk for plain text insertion
para.innerHTML = text;
```

### Use `fetch()`, not `XMLHttpRequest`

```javascript
// GOOD
const response = await fetch("/api/data");

// DEPRECATED
const xhr = new XMLHttpRequest();
```

### Use `console.log()` / `console.error()`, not `alert()`

```javascript
// GOOD
console.log("Debug info");
console.error("Something failed");

// BAD
alert("Debug info");
```

### Don't use browser-prefixed APIs when standard versions exist

```javascript
// GOOD
const context = new AudioContext();

// BAD
const context = new (window.AudioContext || window.webkitAudioContext)();
```

---

## Code Formatting

### Use an automated formatter (Prettier)

Let tooling handle indentation, spacing, and line wrapping to avoid stylistic debates.

### Keep lines short enough to avoid horizontal scrolling

Break long lines at natural points. For long strings, prefer template literals.

### Keep code blocks focused

Show only the code relevant to the concept being demonstrated. For longer examples, link to a complete version.

---

## `const` Mutation Caveat

`const` prevents reassignment, not mutation. Object properties and array contents can still change:

```javascript
const config = { debug: false };
config.debug = true;      // allowed — mutation
config = {};              // TypeError — reassignment

const list = [1, 2, 3];
list.push(4);             // allowed — mutation
list = [];                // TypeError — reassignment
```

---

## Quick Reference

| Aspect | Recommended | Avoid |
|---|---|---|
| Variables | `const`, `let` | `var` |
| Functions | declarations, arrows for callbacks | expressions assigned to identifiers |
| Strings | template literals (with substitution) | concatenation |
| Arrays | literals, `push()`, `for...of` | constructors, index assignment, classical `for` |
| Objects | literals, classes, shorthand | constructors, prototypes |
| Equality | `===`, `!==` | `==`, `!=` |
| Type conversion | `String()`, `Number()` | `"" + val`, `+val` |
| Loops | `for...of`, `forEach()`, semantic methods | `for(;;)`, `for...in` on arrays |
| Async | `async`/`await` | raw promise chains, callbacks |
| Errors | `try...catch` recoverable errors | swallowing exceptions |
| DOM | `textContent`, `fetch()` | `innerHTML` for text, `XMLHttpRequest` |
| Comments | `//` | `/* */` for regular comments |
