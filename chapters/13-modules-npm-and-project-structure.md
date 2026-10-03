# 13. Modules, npm, and project structure

[Back to notes index](../README.md)

| [Previous: Fetch and REST APIs](12-fetch-and-rest-apis.md) | [Notes index](../README.md) | [Next: Prototypes, this, and classes](14-prototypes-this-and-classes.md) |
|:--|:--:|--:|

## Why split code into modules

A module is a file with its own scope that can export values and import values from other modules. Modules help divide an application by responsibility and make dependencies visible.

Browser JavaScript can use modules with a script element whose type is module.

~~~html
<script type="module" src="./src/main.js"></script>
~~~

A module runs in strict mode and its top-level declarations do not become properties on the global window object. Browser modules are deferred until the document has been parsed. They can import other modules using relative paths.

## Export and import named values

A named export identifies each exported value by name.

~~~js
// src/notes/note-utils.js
export function countPending(notes) {
  return notes.filter((note) => !note.reviewed).length;
}

export const maximumTitleLength = 80;
~~~

Import named values with the same exported names. The file extension is included for native browser modules.

~~~js
// src/main.js
import { countPending, maximumTitleLength } from './notes/note-utils.js';

console.log(countPending([]));
console.log(maximumTitleLength);
~~~

Use named exports when a module has several related functions or values. They make the imported API visible at the top of the file.

## Use a default export

A module can have one default export. The importing file can choose its local name.

~~~js
// src/notes/note-store.js
export default class NoteStore {
  constructor() {
    this.notes = [];
  }

  getAll() {
    return [...this.notes];
  }
}
~~~

~~~js
// src/main.js
import NoteStore from './notes/note-store.js';

const store = new NoteStore();
console.log(store.getAll());
~~~

Prefer named exports when they make the module's API easier to scan. Avoid mixing default and named exports without a reason.

## Re-export related values

A module can re-export values from other modules to provide one entry point.

~~~js
export { countPending } from './note-utils.js';
export { default as NoteStore } from './note-store.js';
~~~

Use a re-export file when it improves the public API of a feature. Too many layers of re-exports can make it harder to find where a value is defined.

## Load a module dynamically

Dynamic import loads a module when needed and returns a Promise. It can split optional features from the initial code path.

~~~js
async function openExportTools() {
  const tools = await import('./export-tools.js');
  tools.downloadNotes();
}
~~~

Use dynamic import when loading can wait until a feature is requested. The returned Promise can reject if the file fails to load, so handle failure where the application can respond.

## Use npm to manage a project

npm is a package manager for JavaScript projects. It can create a package manifest, install dependencies, and run named scripts.

~~~powershell
mkdir js-notes-practice
cd js-notes-practice
npm init -y
~~~

A package.json file records project information, scripts, and dependency declarations. A package-lock.json file records the exact dependency tree resolved during installation. Commit both for an application so another install can reproduce the dependency versions.

~~~json
{
  "name": "js-notes-practice",
  "private": true,
  "type": "module",
  "scripts": {
    "test": "node --test"
  }
}
~~~

The type field makes Node.js treat .js files in this package as ECMAScript modules. Without it, Node may use another module mode depending on file extension and configuration. The .mjs extension explicitly marks a module file.

~~~powershell
npm install package-name
npm uninstall package-name
npm test
~~~

The install commands are examples; use a real package name only when the project needs that dependency. npm test runs the test script from package.json. Keep project-specific scripts in that file instead of relying on undocumented commands.

## Understand dependencies and node_modules

A direct dependency is a package the project imports. npm may install additional packages that the direct dependency uses. These packages are stored in node_modules and should not be committed.

~~~gitignore
node_modules/
coverage/
.env
~~~

The package manifest and lock file are committed instead. A fresh checkout can install dependencies with npm install. For automated builds, npm ci installs from the lock file and fails if the manifest and lock file disagree.

Review a package before adding it. Check that it is maintained, has a clear license, fits the project's environment, and solves a problem that is worth adding another dependency for.

## Example project structure

Group files by responsibility and keep file names descriptive.

~~~text
js-notes-practice/
├── package.json
├── package-lock.json
├── index.html
├── src/
│   ├── main.js
│   ├── notes/
│   │   ├── note-store.js
│   │   └── note-utils.js
│   └── ui/
│       └── render-notes.js
└── test/
    └── note-utils.test.js
~~~

A small project does not need a large folder hierarchy. Start with the smallest structure that keeps the entry point, data logic, and view behavior understandable.

## Avoid circular dependencies

If module A imports module B and module B imports module A, the modules form a cycle. Some cycles work, but initialization order can create confusing undefined values or tightly coupled code.

When two modules need each other's details, move the shared value into a third module or change the responsibilities so the dependency flows in one direction.

## Module values are live bindings

An imported name refers to the exported binding, not a copied snapshot. An importing module cannot reassign the imported binding, but it can observe updates made by the exporting module.

~~~js
// counter.js
export let count = 0;

export function increment() {
  count += 1;
}
~~~

~~~js
// main.js
import { count, increment } from './counter.js';

increment();
console.log(count); // 1
~~~

Prefer exporting functions that manage state rather than a mutable exported variable. It gives the module more control over valid changes.

## Hands-on: create two connected modules

1. Create package.json with type set to module.
2. Create src/notes/note-utils.js with a named export.
3. Create src/main.js and import the function using a relative path with the .js extension.
4. Add a script element with type module to index.html.
5. Run the page through a local server.
6. Add a test file and run it with node --test.
7. Install a package only if your project has a real need for it.

## Notes to remember

- Modules keep top-level names scoped to their file.
- Use export and import to define explicit dependencies.
- Include relative file extensions when using native browser modules.
- npm records dependencies and scripts; the lock file records resolved versions.
- Commit package.json and package-lock.json, not node_modules.
- Dynamic import loads a module asynchronously.
- Review a dependency before adding it to a project.

## References

- [MDN JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [MDN import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)
- [MDN export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)
- [Node.js ECMAScript modules](https://nodejs.org/api/esm.html)
- [npm documentation](https://docs.npmjs.com/)
- [npm package.json](https://docs.npmjs.com/cli/configuring-npm/package-json)

---

| [Previous: Fetch and REST APIs](12-fetch-and-rest-apis.md) | [Notes index](../README.md) | [Next: Prototypes, this, and classes](14-prototypes-this-and-classes.md) |
|:--|:--:|--:|
