# compilation-import-attribute

A specification for an [import attribute](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import/with) to differentiate kinds of imports for ECMAScript compilers, bundlers, and runtimes.

## Background

ECMAScript imports can be processed in various environments, including browsers, NodeJS (or similar execution environments), compilers, or bundlers. When preprocessed with compilers or bundlers, ECMAScript imports have historically been configured to behave in at least three distinct ways:

1. *Bundled:* the import is removed in favor of concatenating the imported files together into an output bundle
1. *Code split:* the import is processed as a separate bundle file (often called a "chunk") that is loaded dynamically at runtime, either through native `import()` or, more commonly, through a bundler-specific script load. The chunk itself is compiled as a bundle.
1. *Externalized:* the import is processed at runtime in the browser or NodeJS.

In webpack, rollup, and other popular ECMAScript bundlers, static `import` statements are processed by bundlers to be bundled, by default, whereas code splits are created via dynamic `import()` and externalized imports are configured within the bundler. Non-ECMAScript bundlers, such as esbuild or swc, often follow similar conventions.

## Problem Statement

How an ECMAScript import statement behaves should be understood within the code itself, rather than a combination of the code and bundler configuration. An ECMAScript file should be self contained and portable regardless of which bundler, tooling, or configuration is present within a project.

## Solution

A `compilation` [import attribute](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import/with) allows the software developer to instruct compilers, bundlers, and preprocessors within the code itself, rather than in configuration. This allows the code to function regardless of which tooling, bundler, compiler, or configuration is changed in it. When the `compilation` import attribute is omitted, the bundler or compiler's default behavior is used.

## Spec

Compilers, bundlers, and/or their corresponding plugins support at least the following `compilation` import attributes:

1. `ignore`: the import should be ignored by the compiler and remain intact, to be processed by the runtime execution environment (NodeJS, browser, etc). This is similar to webpack and rollup's `external` option.
1. `bundle`: the import should be removed and the corresponding code should be concatenated in as part of the current bundle. This is similar to webpack and rollup's default behavior for `import` statements
1. `split`: the import should remain intact, but the imported resource should undergo a compilation or bundling process rather than expect the resource to be provided by the runtime execution environment. This is similar to webpack and rollup's processing of dynamic `import()` as well as webpack's SplitChunksPlugin.

## Examples

A bundler, when given the following input:

```js
import { useState } from 'react' with { compilation: "ignore" };
import dayjs from 'dayjs' with { compilation: "bundle" };
import { App } from './App' with { compilation: "split" };
import { createBrowserRouter } from 'react-router';
```

could produce the following output:

```js
import { useState } from 'react';
// dayjs concatenated into the bundle
import { App } from './app-chunk.js';
// react-router concatenated into the bundle, as the default behavior when the compilation import attribute is missing
```

Similarly, an input file with a dynamic import:

```js
// Input
import('react', { with: { compilation: "ignore" } });
```

is compiled to the following:

```js
// Output
import('react');
```

## Benefits

- There is tension between ECMAScript being an interpreted language but a lot of the source code requiring a compilation process before it can be interpreted. The `compilation` import attribute surfaces that tension within the source code rather than burying it within myriad configuration variants across the open source ecosystem.
- Any ECMAScript source code file could be statically analyzed for the `compilation` import attribute in order to know whether it is intended to be executed directly or first pre-processed by a compiler or bundler.
- Webpack's `import(/* webpackIgnore: true */)` special syntax for dynamic imports could be superseded by a syntax not tied to any specific bundler

## Support

To support the `compilation` attribute, the following tooling will be created:

- webpack plugin
- rollup plugin
- babel plugin
- NodeJS loader
- something to make tsc understand it
- any tooling for esbuild, swc, vite, jest, etc

Adding browser support for the `compilation` attribute would likely not be possible, as it is the runtime execution environment rather than a compiler or bundler
