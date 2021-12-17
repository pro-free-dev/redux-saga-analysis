# redux-saga 源码分析

项目中使用到了`redux-saga`，看着文档各种`generator`和`helpers`函数一头雾水，为加深自己的理解，因而觉得有必要深入源码，了解下它的实现方式。

先看概览图，看一看它的全貌，本文主要从以下三部分进行分析：
1. `sagaMiddleware`
2. `runSaga`
3. `sagaHelpers`

<img src='https://user-images.githubusercontent.com/13233825/146486378-9c45dc55-2b65-4c65-9c8f-cdd1ff7fc26e.png' width='800px'>


## 1.sagaMiddleware
官方文档中，引入`saga`中间件，示意代码如下：

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




`const sagaMiddleware = createSagaMiddleware()` 这行代码，是中间件的引入，一切从这里开始。它是`sagaMiddlewareFactory`函数的返回对象，并且在返回前给`sagaMiddleware`挂载了`runSaga`方法。`sagaMiddlewareFactory`里做了以下3件事情：
1. 定义sagaMiddleware函数，并且返回这个函数
2. sagaMiddleware.run，添加`run`函数，用于将`generator`函数添加channel和forkQueue中，在后续dispatch时响应函数
3. sagaMiddleware.setContext，**没有使用到，待研究**

### `sagaMiddleware`函数
直接贴上这个函数的代码，解释直接写在注释中
```js
function sagaMiddleware({ getState, dispatch }) {
    // 将 runSaga 函数指向函数变量 boundRunSaga
    // null: this为函数runSaga自身
    // {}: 绑定参数
    boundRunSaga = runSaga.bind(null, {
      ...options,
      context,
      channel,
      dispatch,
      getState,
      sagaMonitor,
    })

    // 标准的 redux 中间件函数写法
    return next => action => {
      if (sagaMonitor && sagaMonitor.actionDispatched) {
        sagaMonitor.actionDispatched(action)
      }
      const result = next(action) // hit reducers

      // 从这里可以看到，是在reducer之后才会触发saga监听的action
      // 问题1：如果reducer中改变了数据，而saga函数中想获得改变前的数据时，如何获取呢？
      // 问题2：channel 什么时候赋值的呢？
      channel.put(action)
      return result
    }
  }
```

> 问题1：如果reducer中改变了数据，而saga函数中想获得改变前的数据时，如何获取呢？
- 方法1：可以将修改state的代码放入saga函数中，reducer里只做返回新数据对象。比较推荐这种写法，可以将业务代码都限定在saga中。
- 方法2：自己实现中间件，将数据保存在action或context中。
> 问题2：channel 什么时候赋值的呢？
- runSaga中会将监听的action添加至channel中

## 2.runSaga
```js
// run: 是上面的 boundRunSaga 函数，是sagaMiddleware返回时挂载的上来的
// helloSaga: 业务逻辑中写的generator函数
sagaMiddleware.run(helloSaga)

// runSga函数在源码的 ./src/internal/runSaga.js 文件中
```

下面还是贴源码加注释的方式进行解读，对源码进行了部分删减，我们关注核心部分，即`runSaga`做了哪些事情
```jsx
// 入参有三个
// 参数1：sagaMiddlewareFactory中bindRunSaga时已经赋值，都是一些默认参数，也可以自定义传入
// 参数2：核心，我们写的saga函数
// 参数3：说明在run(saga,...otherArguments)时我们还可以传入 otherArguments 其他参数
export function runSaga(
  { channel = stdChannel(), dispatch, getState, context = {}, sagaMonitor, effectMiddlewares, onError = logError },
  saga,
  ...args
) {

  // 核心：saga对应的是rootSaga函数，也就是我们的业务saga的入口
  // rootSaga {<suspended>}
  //   [[GeneratorLocation]]: VM155:1
  //   [[Prototype]]: Generator
  //   [[GeneratorState]]: "suspended"
  //   [[GeneratorFunction]]: ƒ* rootSaga()
  //   [[GeneratorReceiver]]: Window
  //   [[Scopes]]: Scopes[2]
  const iterator = saga(...args)

  // 这里如果没有传入effectMiddlewares，那么这个函数执行的是他本身
  // 默认就是执行原函数
  // export const identity = v => v
  let finalizeRunEffect
  if (effectMiddlewares) {
    const middleware = compose(...effectMiddlewares)
    finalizeRunEffect = runEffect => {
      return (effect, effectId, currCb) => {
        const plainRunEffect = eff => runEffect(eff, effectId, currCb)
        return middleware(plainRunEffect)(effect)
      }
    }
  } else {
    finalizeRunEffect = identity
  }

  // channel: 我们写的saga函数是一个task，channel是管理这些task的列表
  // wrapSagaDispatch 的作用是在action上添加 saga 标签，用来判断当前是支持
  // return dispatch(Object.defineProperty(action, SAGA_ACTION, { value: true }))
  // TODO：待理解SAGA_ACTION的作用
  const env = {
    channel,
    dispatch: wrapSagaDispatch(dispatch),
    getState,
    sagaMonitor,
    onError,
    finalizeRunEffect,
  }

  return immediately(() => {
    // 执行 iterator
    const task = proc(env, iterator, context, effectId, getMetaInfo(saga), /* isRoot */ true, undefined)
    return task
  })
```


### 2.1 proc
