---
title: "How to Convert CommonJS Modules to ESM: A Step-by-Step Guide"
seoTitle: "Convert CommonJS to ESM: Simple Steps Guide"
seoDescription: "Learn how to seamlessly convert CommonJS modules to ESM with this step-by-step guide, addressing common challenges and best practices"
datePublished: Wed May 07 2025 21:57:50 GMT+0000 (Coordinated Universal Time)
cuid: cmaehbxqb000y08kz6m83eyq2
slug: how-to-convert-commonjs-modules-to-esm-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1746654979223/87740a5e-e3a6-405c-85fd-c11f663f8334.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1746655057327/aeeb27e7-e836-4393-b0fb-d5209fbbfc66.png
tags: browser, javascript, browsers, nodejs, commonjs, conversion, esm

---

As JavaScript evolves, ECMAScript Modules (ESM) have become the modern standard for writing modular code. ESM supports native imports in browsers, allows for asynchronous loading, and enables better tree-shaking during bundling. If you're transitioning from CommonJS (require, module.exports), this guide shows how to update your modules and avoid common pitfalls.

---

## Converting `require` to `import`

### Named Imports

**Before (CommonJS):**

```js
const { readFileSync } = require('fs');
const { myFunction1, myFunction2} = require('./myModule');
```

**After (ESM):**

```js
import { readFileSync } from 'fs'; 

import { myFunction } from './myModule.js';
import { myFunction1, myFunction2} from './myModule.js';
```

---

## Converting `module.exports` and `exports` to `export`

### Default Exports

**Before (CommonJS):**

```js
module.exports = function sayHello() {
  console.log('Hello');
};
```

**After (ESM):**

```js
export default function sayHello() {
  console.log('Hello');
}
```

---

### Named Exports

**Before (CommonJS):**

```js
exports.greet = function () {
  console.log('Hi');
};
exports.farewell = function () {
  console.log('Bye');
};
```

**After (ESM):**

```js
export function greet() {
  console.log('Hi');
}

export function farewell() {
  console.log('Bye');
}
```

---

## Importing CommonJS Modules in ESM

If you're working in ESM and need to load a CommonJS module (like many older npm packages), use dynamic `import()`:

```js
const lodash = await import('lodash');
```

> **Note**: Top-level `await` requires Node.js v16+ and must be in an ESM module (`.mjs` or with `"type": "module"` in your `package.json`).

---

## Handling Mixed Exports (Default + Named)

Some libraries export both a default and named exports (e.g., `react` or `chalk`):

```js
import chalk, { red, bold } from 'chalk';
```

If you're unsure what's being exported, check the library's documentation or inspect it:

```js
const chalk = await import('chalk');
console.log(Object.keys(chalk)); // See what’s available
```

---

## How to Enable ESM in Your Project

### Option 1: Use `.mjs` Files

Rename your files to use the `.mjs` extension:

```bash
mv index.js index.mjs
```

Then use ESM syntax directly.

---

### Option 2: Update `package.json`

Add `"type": "module"` to your `package.json`:

```json
{
  "type": "module"
}
```

Now you can use `.js` with ESM imports/exports.

> **Note**: Once this is set, all `.js` files in your project are interpreted as ESM. If you need CommonJS elsewhere (e.g., in config files), rename them to `.cjs`.

---

### Combinations of CommonJS and ESM

### 1\. `module.exports = { a, b }`

```js
// CommonJS
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }
module.exports = { add, subtract };

// ESM
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
```

---

### 2\. `module.exports = function`

```js
// CommonJS
module.exports = function greet(name) {
  return `Hello, ${name}`;
};

// ESM
export default function greet(name) {
  return `Hello, ${name}`;
}
```

---

### 3\. `exports.a = ..., exports.b = ...`

```js
// CommonJS
exports.trim = str => str.trim();
exports.upper = str => str.toUpperCase();

// ESM
export const trim = str => str.trim();
export const upper = str => str.toUpperCase();
```

---

### 4\. `module.exports = { default: fn, x: y }`

```js
// CommonJS
function core() {}
function helper() {}
module.exports = { default: core, helper };

// ESM
export default function core() {}
export function helper() {}
```

---

### 5\. Mutated `module.exports`

```js
// CommonJS
module.exports = {};
module.exports.start = () => {};
module.exports.stop = () => {};

// ESM
export function start() {}
export function stop() {}
```

---

### 6\. Class Export

```js
// CommonJS
class Logger {
  info(msg) { console.log(msg); }
}
module.exports = Logger;

// ESM
export default class Logger {
  info(msg) { console.log(msg); }
}
```

---

### 7\. Named Constants + Default

```js
// CommonJS
const BASE_URL = 'https://api.com';
function fetchData() {}
module.exports = {
  default: fetchData,
  BASE_URL,
};

// ESM
const BASE_URL = 'https://api.com';
function fetchData() {}
export default fetchData;
export { BASE_URL };
```

### 8\. A Single Export for Everything

```js

function doSomething() {
  console.log('Default function');
}

function helperOne() {
  console.log('Helper one');
}

function helperTwo() {
  console.log('Helper two');
}

// CommonJs
module.exports = {
  default: doSomething,
  helperOne,
  helperTwo
};

// ESM
export { doSomething as default, helperOne, helperTwo };
// or written:
export default doSomething;
export { helperOne, helperTwo };
```

### 9\. Hybrid Module (Works with ESM and CommonJS)

While it's technically possible to write code that works in both module systems, it's rarely recommended for production code. Here's an example:

```js
const BASE_URL = 'https://api.example.com';

function get(endpoint) {
  return fetch(`${BASE_URL}/${endpoint}`).then(res => res.json());
}

function post(endpoint, data) {
  return fetch(`${BASE_URL}/${endpoint}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  }).then(res => res.json());
}

const apiClient = { get, post };

export { get, post, BASE_URL };
export default apiClient;

if (typeof module !== 'undefined' && module.exports) {
  module.exports = apiClient;
  module.exports.default = apiClient;
  module.exports.get = get;
  module.exports.post = post;
  module.exports.BASE_URL = BASE_URL;
}
```

> **Important**: This approach has limitations and won't work reliably across all environments. The `typeof module !== 'undefined'` check can be problematic with modern bundlers and may not behave as expected in browser environments.

**Better alternatives:**

1. **Use a bundler** — Tools like Webpack, Rollup, or esbuild can generate proper dual-format builds (ESM + CommonJS) from a single ESM source. This is the recommended approach for libraries.
    
2. **Publish dual formats** — Write your code in ESM, then use a build tool to output both ESM and CommonJS versions:
    
    ```json
    {
      "type": "module",
      "main": "./dist/index.cjs",
      "module": "./dist/index.js",
      "exports": {
        "import": "./dist/index.js",
        "require": "./dist/index.cjs"
      }
    }
    ```
    
3. **ESM-only** — For new projects, consider going ESM-only. Most modern environments support it, and you can always use dynamic `import()` in CommonJS if needed.
    

**When the hybrid approach might make sense:**

* Quick prototypes or internal tools
    
* Educational examples
    
* Simple utility files you're copying between projects
    

For anything you're publishing to npm or using in production, stick with proper tooling to create a true Universal Module Definition (UMD) build or dual-format distribution. Modern bundlers handle the complexity of detecting the runtime environment (CommonJS, AMD, or browser globals) and adapting accordingly, without the fragility of manual runtime checks.

## Common Pitfalls

Here are common issues developers run into when migrating:

### Missing File Extensions

ESM requires full paths including file extensions.

```js
// Incorrect
import myUtil from './util'; 

// Correct
import myUtil from './util.js';
```

### Mixing `require` with `import`

Don’t mix module systems in the same file. Pick one, ideally ESM for new code.

### Forgetting Top-Level `await` Limitations

Only available in ESM files. If using top-level await, ensure you're inside an ESM module or wrap the code in an async function.

---

I hope this can clear up any confusion for returning developers who just needed a primer to peruse or new developers trying to clear up their confusion as they navigate older code bases.

## Resources

* [Node.js ESM Documentation](https://nodejs.org/api/esm.html)
    
* [MDN JavaScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)