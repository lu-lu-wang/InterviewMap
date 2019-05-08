[Rematch](https://rematch.gitbook.io/handbook/ling-gan)

### Rematch没有多余的action types、action creators、switch语句或者thunks

#### 快速开始
> npm install @rematch/core

### 第一步 **init**
index.js

```
  import { init } from '@rematch/core'
  import * as models from './models'
  export default store = init({
    models
  })

```
### 第二步 **Models**
#### 该model促使state、reducers、async actions和action creators放在同一个地方。
models.js

```
  export const count = {
    state: 0,
    reducers: {
      increment(state, payload){
        return state + payload
      }
    },
    effects: {
      async incrementAsync(payload,rootState){
        await new Promise(resolve => setTimeout(resolve, 1000))
        this.increment(payload)
        // dispatch.count.increment(payload)
      }
    }
  }

```
#### 理解模型与回答几个问题一样简单
1. 我的初始化state是什么？ state
2. 我该如何改变state? reducers
3. 我该如何处理异步action？ effects with async/await

### 第三步 **Dispatch**
#### dispatch是我们如何在你的model中触发reducers和effects，Dispatch标准化了你的action，而无需编写action types或者action creators

```
import { diaptch } from '@reamtch/core'

// reducers

dispatch({type:'models/method',payload})
dispatch.model.method(payload)

// effects
dispatch({type:'models/method',payload})
dispatch.model.method(payload)

**Dispatch可以直接被调用，或者用dispatch[method][action](payload)简写**

```
### 第四步 **View**
* count: JS|React|Vue|Angular
* Todos: React

```
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider, connect } from 'react-redux'
import store from './index'

const mapState = state => ({
  count: state.count
})
const mapDispatch = ({ count: { increment, incrementAsync }}) => ({
  increment: () => increment(1),
  incrementAsync: () => incrementAsync(1)
})
const Count = props => (
  <div>
    {props.count}
    <button onClick={props.increment}>increment</button>
    <button onClick={props.incrementAsync}>incrementAsync</button>
  </div>
)
const CountContainer = connect(mapState,mapDispatch)(Count)
ReactDOM.render(
  <Provider store={store}>
    <CountContainer/>
  </Provider>,
  document.getElementById('root')
)

```
### 从**Redux**迁移
#### 从Redux转到Rematch只需几步：
1. 基于Redux启动Rematch init
2. 合并reducers和models
3. 转移到模型

## **目的**
Redux是一个出色的**状态管理工具**，有健全的中间件生态与出色的开发工具。
Rematch在Redux的基础上构建并减少了样板代码和执行了一些最佳实践。

Rematch移除了Redux所需要的这些东西：
* 声明action类型
* action创建函数
* thunks
* store配置
* mapDispatchToProps
* sagas

### **Redux** 与 **Rematch** 的对比

**Rematch**
1. model

```
import { init } from '@rematch/core'

const count = {
  state: 0,
  reducers: {
    upBy: (state, payload) => state + payload
  }
}
init({
  models: { count }
})

```
2. View

```
import { connect } from 'react-redux'

// Component

const mapStateToProps = (state) => ({
  count: state.count
})
const mapDispatchToProps = (dispatch) => ({
  countUpBy: dispatch.count.upBy
})
connect(mapStateToProps,mapDispatchToProps)(Component)

```
**Redux**
1. store

```
import { createStore, combineReducers } from 'redux'
// devtools,reducers,middleware,etc
export default createStore(reducers,initialState,enhancers)

```
2. Action Type

```
export const COUNT_UP_BY = 'COUNT_UP_BY'

```
3. Action Creator

```
import { COUNT_UP_BY } from '../types/counter'
export const countUpBy = (value) => ({
  type: COUNT_UP_BY,
  payload: value
})

```
4. Reducer

```
import { COUNT_UP_BY } from '../types/counter'
const initialState = 0

export default (state = initialState, action) => {
  switch (action.type) {
    case COUNT_UP_BY:
      return state + action.payload
    default: return state
  }
}
```
5. view

```
  import { countUpBy } from '../actions/count'
  import { connect } from 'react-redux'

  // Component

  const mapStateToProps = (state) => ({
    count: state.count
  })
  connect(mapStateToProps, { countUpBy })(Component)

```
## **API** 

## rematch

### @rematch/core API
> import { init, dispatch, getState } from '@rematch/core'

### init
> init(config)
该函数被调用去设置Rematch, 返回store

``` 
import { init } from '@rematch/core'

const store = init()

```
init也可以通过下面配置来调用

### models
> init({ models: { [string]: model } })

```
import { init } from '@rematch/core'

const count = {
  state: 0
}
init({
  models: {
    count
  }
})

```
> 对于较小的项目我们可以直接在models.js文件中直接将model导出，对于较大的项目我们可以将model保存在一个文件也中，并且导出。

```
// models/count.js
export default {
  state: 0
}

// models/index.js

export { default as count } from './count'
export { default as settings } from './setting'

**store中直接初始化**
import { init } from '@rematch/core'
import * as models from './models'
init({ models })

```
### state
> state: any(初始化model中的state)

```
const example = {
  state: {loading: false}
}
```
### reducers
> reducers: { [string]: (state, payload) => any }
一个改变该models state的所有函数的对象。这些函数采用model的上一次state和一个payload作为形参，并且返回model的一个状态。这些仅依赖于state和payload参数来计算下一个state的纯函数。对于依赖副作用函数要使用effects

```
  {
    reducers: {
      add: (state, payload) => state + payload
    }
  }
```
>通过列出model+action来作为key，Reducers也可以监听来自其他model的action。
```
{
  reducers: {
    'otherModel/actionName': (state, payload) => state + payload
  }
}
```
### effects
> effects: {[string]:(payload,rootState)} 可处理(所有)副作用函数的对象
```
{
  effects: {
    logState(payload, rootState){
      console.log(rootState)
    }
  }
}
```
>与async/await一起使用时，Effects提供了一种处理异步action的简单方法

```
{
  effects: {
    async loadData(payload, rootState) {
      // wait for data to load
      const response = await fetch('http://example.com/data')
      const data = await response.json()
      // pass the result to a local reducer
      dispatch.example.update(data)
    }
  }
}
```
### plugins
> init({plugins: [loadingPlugin, persistPlugin]})
Plugins用来自定义init配置或内部hooks，它能添加功能到你的rematch设置中

## redux

```
init({
  redux: {
    middleware: [reduxLogger],
    reducers: {
      someReducers: {
        someReducer: (state, action)=> ...
      }
    }
  }
})
```
需要直接访问redux的情况：
* 迁移现有的Redux项目
* 添加中间件
* 创建一个自定义插件

### store
> store.dispatch
在Redux中，一个分派action的函数
在Rematch中，store.dispatch能被直接调用或者作为一个对象
```
import store from './index'

const { dispatch } = store

// reducers
dispatch({ type: 'count/increment', payload})
dispatch.count.increment()
```
### store.getState
>在Redux中，返回该store的state
### store.name
>为store提供一个名称，当使用多个store时使用。当全局getState被调用时这个名字将变成key
### store.model
>调用init之后，延迟加载model并将它们合并到rematch中，使用store.model.
```
import { init } from '@rematch/core'

const store = init({
  models: {
    count: { state: 0 }
  }
})
store.getState() // { count: 0 }
store.model({ name: 'countB', state: 99 })
store.getState()
// { count: 0, countB: state: 99 }
```
### dispatch
> dispatch(action,meta)
在所有store中，Dispatch发送并触发action
> dispatch.modelName.actionName(any)
Dispatch具有可选的第二个属性mata，它可以用于subscriptions或middleware
> dispatch.cart.addToCart(item, { syncWithServer: true })

### action
> { type: 'modelName/actionName',payload: any }
Action是在Redux中发送的消息，作为应用程序的不同部分传递状态更新的一种方式。
在Rematch中，一个action时钟是一个model+action类型的结构
### getState
> getState(): { [storeName]: state }
返回一个包含所有store state的对象

```
import { init, getState } from '@rematch/core'

const firstStore = init({
  name: 'first',
  models: { count: { state: 0 } }
})

const secondStore = init({
  name: 'second',
  models: { count: { state: 5 }}
})

getState() // { first: { count: 1 }, second: { count: 5 } }

```
## 技巧 
### Devtools
Rematch具有开箱即用的Redux Devtools，不需要配置
> init() // devtools up and running
还可添加redux ddevtools
```
init({
  redux: {
    devtoolOptions: options
  }
})
```
## React

```
import React from 'react'
import { connect } from 'react-redux'

const mapState = (state) => ({
  count: state.count
})
const mapDispatch = (props) => ({
  // 将dispatch语句保留在mapDispatch中，保持组件纯粹，易于测试
  increment: () => dispatch.count.increment()
})

const Counter = (props) => (
  <div>
    {props.count}
    <button onClick={props.increment}/>
  </div>
)
// 解构
const mapDispatchWithDestructure = ({count:{increment}})=>({increment})

export default connect(mapState,mapDispatch)(Counter)
```
> 使用store来设置你的React-Redux Provider

```
import React from 'react'
import { Provider } from 'react-redux'
import { init } from '@rematch/core'
import App from './App'

const store = init()

export default () => (
  <Provider store={store}>
    <App />
  </Provider>
)
```

#比较
兼容性| Rematch | Mirror | dva
--- | --- | --- | ---
适用框架 | 所有框架/不使用框架 | React | React
适用路由 | 所有路由/不使用路由 | RR4 | RR3,RR4/不使用路由
移动端 | 适用 | 不适用 | 适用
开发者工具 | Redux,Reactotron | Redux | Redux
插件化 | 适用 | 适用 | 适用
reducers | 适用 | 适用 | 适用
effects | async/await | async/await | redux saga
effects params | (payload,internals) | (action,state) | (action,state)
监听方式 | subscriptions | hooks | subscriptions
懒加载模型 | 适用 | 适用 | 适用
链式dispatch | 适用 | 适用 | 适用
直接dispatch | 适用 | 不可 | 不可
dispatch promises | 适用 | 不可 | 不可
加载插件 | 适用 | 适用 | 适用
presist plugin | 适用 | 不可 | 不可

### 从Redux迁移
从Redux迁移到Rematch只会在状态管理上有微小的改变，不会影响到你的视图逻辑，可通过现有的reducers当做extraReducers传入init继续使用你当前的reducers，你也可使用dispatch直接出发actions
### 可组合的插件
Rematch从里到外都是构造于插件机制上。无论是dispatch还是selectors统统都是插件，因此，你可以创造一个复杂的自定义插件，在modal上修改setup或者add，而无需对Rematch本身做任何修改。