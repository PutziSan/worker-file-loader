<div align="center">
  <a href="https://github.com/webpack/webpack">
    <img width="200" height="200" src="https://webpack.js.org/assets/icon-square-big.svg">
  </a>
</div>

[![npm][npm]][npm-url]
[![node][node]][node-url]
[![deps][deps]][deps-url]
[![tests][tests]][tests-url]
[![chat][chat]][chat-url]
[![size][size]][size-url]

# worker-file-loader

worker loader module for webpack

> this module is forked from [worker-loader](https://github.com/webpack-contrib/worker-loader) and customized so that the url of the created worker is returned to create service-workers or workers with the url.

## Requirements

This module requires a minimum of Node v6.9.0 and Webpack v4.0.0.

## Getting Started

To begin, you'll need to install `worker-file-loader`:

```console
$ npm install worker-file-loader --save-dev
```

## `window is not defined`-error in development

This is [a known error](https://github.com/webpack/webpack/issues/6642) and eventually fixed, when the `target: "universal"`-feature lands in webpack ([#6525](https://github.com/webpack/webpack/issues/6525)).

To fix the error in the meanwhile, you can add this to your webpack-config:

```javascript
const isDevelopment = process.env.NODE_ENV === 'development';
module.exports = {
  // ...
  output: {
    // ...
    globalObject: isDevelopment
      ? "(typeof self !== 'undefined' ? self : this)"
      : undefined,
  },
};
```

### Inlined

```js
// App.js
import Worker from 'worker-file-loader!./Worker.js';
```

### Config

```js
// webpack.config.js
{
  module: {
    rules: [
      {
        test: /\.worker\.js$/,
        use: { loader: 'worker-file-loader' },
      },
    ];
  }
}
```

```js
// App.js
import workerUrl from './file.worker.js';

const worker = new Worker(workerUrl);

worker.postMessage({ a: 1 });
worker.onmessage = function(event) {};

worker.addEventListener('message', function(event) {});
```

And run `webpack` via your preferred method.

## Options

### `fallback`

Type: `Boolean`
Default: `false`

Require a fallback for non-worker supporting environments

```js
// webpack.config.js
{
  loader: 'worker-file-loader';
  options: {
    fallback: false;
  }
}
```

### `name`

Type: `String`
Default: `[hash].worker.js`

To set a custom name for the output script, use the `name` parameter. The name
may contain the string `[hash]`, which will be replaced with a content dependent
hash for caching purposes. When using `name` alone `[hash]` is omitted.

```js
// webpack.config.js
{
  loader: 'worker-file-loader',
  options: { name: 'WorkerName.[hash].js' }
}
```

### publicPath

Type: `String`
Default: `null`

Overrides the path from which worker scripts are downloaded. If not specified,
the same public path used for other webpack assets is used.

```js
// webpack.config.js
{
  loader: 'worker-file-loader';
  options: {
    publicPath: '/scripts/workers/';
  }
}
```

## Examples

The worker file can import dependencies just like any other file:

```js
// Worker.js
const _ = require('lodash');

const obj = { foo: 'foo' };

_.has(obj, 'foo');

// Post data to parent thread
self.postMessage({ foo: 'foo' });

// Respond to message from parent thread
self.addEventListener('message', (event) => console.log(event));
```

### Integrating with ES2015 Modules

_Note: You can even use ES2015 Modules if you have the
[`babel-loader`](https://github.com/babel/babel-loader) configured._

```js
// Worker.js
import _ from 'lodash';

const obj = { foo: 'foo' };

_.has(obj, 'foo');

// Post data to parent thread
self.postMessage({ foo: 'foo' });

// Respond to message from parent thread
self.addEventListener('message', (event) => console.log(event));
```

### Integrating with TypeScript

To integrate with TypeScript, you will need to define a custom module for the exports of your worker

```typescript
// typings/custom.d.ts
declare module 'worker-file-loader!*' {
  const url: string;

  export default url;
}
```

```typescript
// Worker.ts
const ctx: Worker = self as any;

// Post data to parent thread
ctx.postMessage({ foo: 'foo' });

// Respond to message from parent thread
ctx.addEventListener('message', (event) => console.log(event));
```

```typescript
// App.ts
import workerUrl from 'worker-file-loader!./Worker';

const worker = new Worker(workerUrl);

worker.postMessage({ a: 1 });
worker.onmessage = (event) => {};

worker.addEventListener('message', (event) => {});
```

### Cross-Origin Policy

[`WebWorkers`](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)
are restricted by a
[same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy), so if
your `webpack` assets are not being served from the same origin as your
application, their download may be blocked by your browser. This scenario can
commonly occur if you are hosting your assets under a CDN domain. Even downloads
from the `webpack-dev-server` could be blocked. There are two workarounds:

Firstly, you can inline the worker as a blob instead of downloading it as an
external script via the [`inline`](#inline) parameter (**this option is removed for this loader**)

Secondly, you may override the base download URL for your worker script via the
[`publicPath`](#publicpath) option

```js
// App.js
// This will result in `/workers/file.worker.js`
import workerUrl from './file.worker.js';
```

```js
// webpack.config.js
{
  loader: 'worker-file-loader';
  options: {
    publicPath: '/workers/';
  }
}
```

## Contributing

Please take a moment to read our contributing guidelines if you haven't yet done so.

#### [CONTRIBUTING](./.github/CONTRIBUTING.md)

## License

#### [MIT](./LICENSE)

[npm]: https://img.shields.io/npm/v/worker-loader.svg
[npm-url]: https://npmjs.com/package/worker-loader
[node]: https://img.shields.io/node/v/worker-loader.svg
[node-url]: https://nodejs.org
[deps]: https://david-dm.org/webpack-contrib/worker-loader.svg
[deps-url]: https://david-dm.org/webpack-contrib/worker-loader
[tests]: https://img.shields.io/circleci/project/github/webpack-contrib/worker-loader.svg
[tests-url]: https://circleci.com/gh/webpack-contrib/worker-loader
[cover]: https://codecov.io/gh/webpack-contrib/worker-loader/branch/master/graph/badge.svg
[cover-url]: https://codecov.io/gh/webpack-contrib/worker-loader
[chat]: https://img.shields.io/badge/gitter-webpack%2Fwebpack-brightgreen.svg
[chat-url]: https://gitter.im/webpack/webpack
[size]: https://packagephobia.now.sh/badge?p=worker-loader
[size-url]: https://packagephobia.now.sh/result?p=worker-loader
