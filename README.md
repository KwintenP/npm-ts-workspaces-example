# How to build TypeScript mono-repo project

## Tools

- npm (v7 or later)
- Lerna: Multiple packages management tool.

## Directory Structure

Put each package under the `packages` directory.

```
.
├── node_modules/
├── README.md
├── lerna.json
├── package-lock.json
├── package.json
├── packages
│   ├── x-cli
│   │   ├── lib
│   │   │   ├── cli.d.ts
│   │   │   ├── cli.js
│   │   │   ├── cli.js.map
│   │   │   ├── main.d.ts
│   │   │   ├── main.js
│   │   │   └── main.js.map
│   │   ├── package.json
│   │   ├── src
│   │   │   ├── cli.ts
│   │   │   └── main.ts
│   │   └── tsconfig.json
│   └── x-core
│       ├── lib
│       │   ├── index.d.ts
│       │   ├── index.js
│       │   └── index.js.map
│       ├── package.json
│       ├── src
│       │   └── index.ts
│       └── tsconfig.json
├── tsconfig.build.json
└── tsconfig.json
```

## Workspaces

Using [NPM workspace feature](https://github.com/npm/rfcs/blob/latest/implemented/0026-workspaces.md), configure the following files:

- /package.json

Append the `workspaces` key.

```json
{
  "private": true,
  "workspaces": ["packages/*"]
}
```

- lerna.json

Set `npmClient` `"yarn"` and turn `useWorkspaces` on.

```json
{
  "lerna": "2.2.0",
  "packages": ["packages/*"],
  "npmClient": "yarn",
  "useWorkspaces": true,
  "version": "1.0.0"
}
```

Exec `yarn install`(or `lerna bootstrap`). After successful running, all dependency packages are downloaded under the repository root `node_modules` directory.

### Dependencies between packages

In this example, the `x-cli` package depends on another package, `x-core`. So to execute (or test) `x-cli`, `x-core` packages should be installed.
But in development the `x-core` package is not published so you can't install it.

`yarn` solves this problem. This command creates sim-links of each package into the top-level `node_modules` dir.

## Resolve Dependencies as TypeScript Modules

As mentioned above, Lerna resolves dependencies between packages. It's enough for "runtime". However considering TypeScript sources, in other words "static", it's not.

For example, the following code depends a module `x-core` located at other package.

```ts
/* packages/x-cli/src/main.ts */
import { awesomeFn } from '@quramy/x-core';

export function cli() {
  awesomeFn();
  return Promise.resolve(true);
}
```

If you compile this code, TypeScript compiler emits a "Cannot find module" error until building `x-core` package and creating `x-core/index.d.ts`. And it's silly to compile dependent packages(e.g. `x-core`) in the same repository after each editing them.

[TypeScript's path mapping](https://www.typescriptlang.org/docs/handbook/module-resolution.html#path-mapping) is the best solution. Path mappings are declared such as:

```js
/* tsconfig.json */
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "baseUrl": "./packages",
    "paths": {
      "@quramy/*": ["./*/src"]
    }
  }
}
```

The above setting means `import { awesomeFn } from "@quramy/x-core"` is mapped to `import { awesomeFn } from "../../x-core/src"`(it's relative from "packages/x-cli/src/main.ts"). In other words, path mapping allows to treat developing packages' sources as published(compiled) modules.

## License

The MIT License (MIT)

Copyright 2017 Quramy

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
