# JavaScript

```javascript
// reference a variable between different js files
// first export in Interpolation.js
export const berechnungPerPlusEins = i9date.add(berechnungPer, 1, 0, 0)
// in other file
import { berechnungPerPlusEins } from './Interpolation.js'
```

An object in Java Script is can be expressed in three different ways:

- `new Object()`
- `Object.create()`
- an [object initializer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer?retiredLocale=de), e.g. `const object1 = { a: 'foo', b: 42, c: {} }`

```javascript
const object1 = { a: 'foo', b: 42, c: {} };

console.log(object1.a);
// expected output: "foo"

const a = 'foo';
const b = 42;
const c = {};
const object2 = { a: a, b: b, c: c };

console.log(object2.b);
// expected output: 42

const object3 = { a, b, c };

console.log(object3.a);
// expected output: "foo"
```

- projects can be generated with `npm`.

## Module structure

In Javascript, there are two module standards: Node.js uses another module standard called `commonjs` in contrast to `es modules` or just `modules` used by browsers. The differences can be seen [here](https://nodejs.org/api/esm.html#esm_differences_between_es_modules_and_commonjs). As we seem to use the `commonjs` standard in oc4 (we have classes like `CommonJSSupport`), we will stick with `CommonJS`. Otherwise, we would have to make jest automatically accept `es modules` by calling the test method via `node --experimental-vm-modules node_modules/jest/bin/jest.js`. The flag is [experimental](https://jestjs.io/docs/ecmascript-modules).
