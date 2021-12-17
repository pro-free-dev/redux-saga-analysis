# redux-saga 源码分析

项目中使用到了`redux-saga`，看着文档各种`generator`和`helpers`函数，为加深自己的理解，因而决得有必要深入源码，了解下它的实现方式。

先看概览图，看一看它的全貌，本文主要从以下三部分进行分析：
1. `sagaMiddleware`
2. `runSaga`
3. `sagaHelpers`

<img src='https://user-images.githubusercontent.com/13233825/146486378-9c45dc55-2b65-4c65-9c8f-cdd1ff7fc26e.png' width='800px'>


## sagaMiddleware
官方文档中，示意代码如下：

```js
// ...
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

// ...
import { helloSaga } from './sagas'

const sagaMiddleware = createSagaMiddleware()
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
sagaMiddleware.run(helloSaga)

const action = type => store.dispatch({type})

// rest unchanged
```





```javascript
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// create the saga middleware
const sagaMiddleware = createSagaMiddleware()
// mount it on the Store
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// then run the saga
sagaMiddleware.run(mySaga)

// render the application
```

# Documentation

- [Introduction](https://redux-saga.js.org/docs/introduction/BeginnerTutorial.html)
- [Basic Concepts](https://redux-saga.js.org/docs/basics/DeclarativeEffects)
- [Advanced Concepts](https://redux-saga.js.org/docs/advanced/Channels)
- [Recipes](https://redux-saga.js.org/docs/recipes/index.html)
- [External Resources](https://redux-saga.js.org/docs/ExternalResources.html)
- [Troubleshooting](https://redux-saga.js.org/docs/Troubleshooting.html)
- [Glossary](https://redux-saga.js.org/docs/Glossary.html)
- [API Reference](https://redux-saga.js.org/docs/api/index.html)

# Translation

- [Chinese](https://github.com/superRaytin/redux-saga-in-chinese)
- [Traditional Chinese](https://github.com/neighborhood999/redux-saga)
- [Japanese](https://github.com/redux-saga/redux-saga/blob/master/README_ja.md)
- [Korean](https://github.com/mskims/redux-saga-in-korean)
- [Portuguese](https://github.com/joelbarbosa/redux-saga-pt_BR)
- [Russian](https://github.com/redux-saga/redux-saga/blob/master/README_ru.md)

# Using umd build in the browser

There is also a **umd** build of `redux-saga` available in the `dist/` folder. When using the umd build `redux-saga` is available as `ReduxSaga` in the window object. This enables you to create Saga middleware without using ES6 `import` syntax like this:

```javascript
var sagaMiddleware = ReduxSaga.default()
```

The umd version is useful if you don't use Webpack or Browserify. You can access it directly from [unpkg](https://unpkg.com/).

The following builds are available:

- [https://unpkg.com/redux-saga/dist/redux-saga.umd.js](https://unpkg.com/redux-saga/dist/redux-saga.umd.js)
- [https://unpkg.com/redux-saga/dist/redux-saga.umd.min.js](https://unpkg.com/redux-saga/dist/redux-saga.umd.min.js)

**Important!** If the browser you are targeting doesn't support *ES2015 generators*, you must transpile them (i.e. with [babel plugin](https://github.com/facebook/regenerator/tree/master/packages/regenerator-transform)) and provide a valid runtime, such as [the one here](https://unpkg.com/regenerator-runtime/runtime.js). The runtime must be imported before **redux-saga**:

```javascript
import 'regenerator-runtime/runtime'
// then
import sagaMiddleware from 'redux-saga'
```

# Building examples from sources

```sh
$ git clone https://github.com/redux-saga/redux-saga.git
$ cd redux-saga
$ yarn
$ npm test
```

Below are the examples ported (so far) from the Redux repos.

### Counter examples

There are three counter examples.

#### counter-vanilla

Demo using vanilla JavaScript and UMD builds. All source is inlined in `index.html`.

To launch the example, open `index.html` in your browser.

> Important: your browser must support Generators. Latest versions of Chrome/Firefox/Edge are suitable.

#### counter

Demo using `webpack` and high-level API `takeEvery`.

```sh
$ npm run counter

# test sample for the generator
$ npm run test-counter
```

#### cancellable-counter

Demo using low-level API to demonstrate task cancellation.

```sh
$ npm run cancellable-counter
```

### Shopping Cart example

```sh
$ npm run shop

# test sample for the generator
$ npm run test-shop
```

### async example

```sh
$ npm run async

# test sample for the generators
$ npm run test-async
```

### real-world example (with webpack hot reloading)

```sh
$ npm run real-world

# sorry, no tests yet
```

### TypeScript

Redux-Saga with TypeScript requires `DOM.Iterable` or `ES2015.Iterable`. If your `target` is `ES6`, you are likely already set, however, for `ES5`, you will need to add it yourself.
Check your `tsconfig.json` file, and the official <a href="https://www.typescriptlang.org/docs/handbook/compiler-options.html">compiler options</a> documentation.

### Logo

You can find the official Redux-Saga logo with different flavors in the [logo directory](https://github.com/redux-saga/redux-saga/tree/master/logo).


## Redux Saga chooses generators over `async/await`

A [few](https://github.com/redux-saga/redux-saga/issues/1373#issuecomment-381320534) [issues](https://github.com/redux-saga/redux-saga/issues/987#issuecomment-301039792) have been raised asking whether Redux saga plans to use `async/await` syntax instead of generators.

We will continue to use [generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator). The primary mechanism of `async/await` is Promises and it is very difficult to retain the scheduling simplicity and semantics of existing Saga concepts using Promises. `async/await` simply don't allow for certain things - like i.e. cancellation. With generators we have full power over how & when effects are executed.

## Backers
Support us with a monthly donation and help us continue our activities. \[[Become a backer](https://opencollective.com/redux-saga#backer)\]

<a href="https://opencollective.com/redux-saga/backer/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/backer/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/backer/29/avatar.svg"></a>


### Sponsors
Become a sponsor and get your logo on our README on Github with a link to your site. \[[Become a sponsor](https://opencollective.com/redux-saga#sponsor)\]

<a href="https://opencollective.com/redux-saga/sponsor/0/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/0/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/1/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/1/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/2/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/2/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/3/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/3/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/4/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/4/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/5/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/5/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/6/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/6/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/7/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/7/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/8/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/8/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/9/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/9/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/10/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/10/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/11/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/11/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/12/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/12/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/13/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/13/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/14/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/14/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/15/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/15/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/16/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/16/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/17/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/17/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/18/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/18/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/19/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/19/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/20/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/20/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/21/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/21/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/22/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/22/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/23/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/23/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/24/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/24/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/25/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/25/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/26/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/26/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/27/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/27/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/28/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/28/avatar.svg"></a>
<a href="https://opencollective.com/redux-saga/sponsor/29/website" target="_blank"><img src="https://opencollective.com/redux-saga/sponsor/29/avatar.svg"></a>

## License
Copyright (c) 2015 Yassine Elouafi.

Licensed under The MIT License (MIT).
