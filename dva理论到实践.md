# Dva简介
目前最流行的数据流方案：

	最流行的React应用架构方案如下：
	路由： React-router
	架构： Redux
	异步操作： Redux-saga
	缺点： 要引入多个库，项目结构复杂
	dva是体验技术部开发的React应用框架，将上面三个React工具库包装在一起，简化了API，让开发React更加方便和快捷。
	dva = react-router + redux + redux-saga
	dva最简单的应用如下所示：
	
	import dva from ‘dva’
	const App = () => <div>hello dva</div>
	
	// 创建应用
	const app = dva()
	// 注册视图
	app.router(() => <App />)
	// 启动视图
	app.start(‘#root’)

1.借鉴elm的概念，Reudcer,effect和Subscription

2.框架而非类库

3.基于redux,react-router,react-saga的轻量级封装

# Dva的特性
1、仅有5个主要的api

2、支持HRM，支持模块的热更新

3、支持SSR，支持服务器端渲染

4、支持Mobile/ReactNative，支持移动手机端的代码编写

5、支持TypeScript，个人感觉这个是js的一个趋势

6、支持路由和Model的动态加载

Dva的5个AIP： 

	import dva from ‘dva’
	const app = dva()    // 1.创建dva实例
	app.use(require(‘dva-loading’))  // 2.装载插件
	app.model(require(‘./models/count’)) // 3.注册Model
	app.router(require(‘./router’)) // 4.配置路由
	app.start(‘#root’) // 5.启动应用
	
	1.app = dva(): 创建应用，返回dva实例。
	
	2.app.use(hooks): 配置hooks或者注册插件
	import createLoading from ‘dva-loading’
	app.use(createLoading(opts))
	
	3.app.model(modelObject):这个是你数据逻辑处理，数据流动的地方。modle是dva里面与我们真正进行项目开发，逻辑处理，数据流动的地方。这里面涉及到的namespace、modle、effects和reducer等概念都很重要。
	
	4.app.router(funciton):注册路由表，我们做路由跳转的地方
	5.app.start([HTMLElemnt],opts),启动应用

# Dva的九个概念
### 数据流向
数据的改变发生通常是通过用户交互行为或者浏览器行为（如路由的跳转等）触发，当此类行为会改变的互数据的时候可以通过dispatch发起一个action，如果是同步行为会直接通过Reducers改变State，如果是一步行为（副作用）会先触发Effects然后流向Reducers最终改变State，所以在dva中，数据流非常清晰。

### Models层
1. State(状态)：
type State = any
State表示Model的状态数据，通常表现为一个js对象（当然它可以是任何值）；操作的时候每次都要当做不可变数据来对待，保证每次都是全新对象，没有引用关系，这样此昂保证State的独立性，便于测试和追踪变化。------在dva中你可以通过dva的实例属性_store看到顶部的state数据： 

	const app = dva()
	console.log(app._store) // 顶部的state数据
	
初始值，我们在dva()初始化的时候和在model里面的state对其两处进行定义，其中modal中的优先级低于传给dva()的opts.initalState

2. Action：
type AsyncAction = any  ->
Action是一个普通的js对象，它是改变State的唯一途径。无论是从UI事件、网络回调，还是WebSocket等数据源获得的数据，最终都会通过dispatch函数调用一个action，从而改变对应的数据，action必须带有type属性指明具体的行为，其它字段可以自定义，如果发起一个action需要使用dispatch函数，需注意dispatch是在组件connect Models以后，通过props传入的。

Action表示操作事件，可以是同步，也可以是异步。action需要一个type，表示这个aciton要触发什么操作，playload则表示action要传递的数据

3. dispatch函数

type dispatch = (a: Action) => Action

dispatch function是一个用于触发action的函数，action是改变State的唯一途径，但是它只描述了一个行为，而dispatch可以看做是触发这个行为的方式，而Reducer则是描述如何改变数据。在dva中，connect Model的组件通过props可以访问到dispatch，可以调用Model中的Reducer或者Effects

4. Reducer
	type Reducer<S, A> = (state: S, action: A) => S
	Reducer(也称为reducing function)函数接受两个参数：之前已经累积运算的结果和当前要被积累的值，返回的是一个新的积累结果。该函数把一个集合归并成一个单值。Reducewr的概念来自于函数值编程，很多语言中都有Reducer API,

		[{x:1},{z:3}].reduce(function(prev, next){
			return Object.assign(prev, next)
		})
在dva中，reducer聚合积累的结果是当前model的state对象，通过actions中传入的值，与当前reducers中的值进行运算获得新的值（也就是新的state）。需要注意的是Reducers必须是纯函数，所以同样的输入必然得到同样的输出，它们不应该产生任何副作用。并且，每一次计算都应该使用immutable data,这种特性简单理解就是每次操作都是返回一个全新的数据，所以热重载和时间旅行这些功能才能使用。

5. Effect

	Effects被称为副作用，在我们的应用中，最常见的就是异步操作，它来自于函数编程的概念，之所以叫做副作用是因为它使得我们的函数变得不纯，同样的输入不一定获得同样的输出。dva为了控制副作用的操作，底层引入了redux-saga做异步控制流程，由于采用了generator相关概念，所以将异步转换成同步写法，从而将effects转为纯函数。

6. Subscription
	Subscription 是一种从源获取数据的方法，它来自于elm

Subscription 语义是订阅，用于订阅一个数据源，然后根据条件dispatch需要的action。数据源可以是当前时间，服务器的websocket连接，keyboard输入，geolocation变化，history变化等。

	import key from ‘keymaster
	...
	app.model({
		namespace: ‘count’,
		subscription: {
			keyEvent({dipatch}) {
				key(⌘+up, ctrl+up,()=>{dispatch({type: ‘add’})})
			}
		}
	})

7. Router
	这里的路由通常是指前端路由，由于我们的应用现在通常是单页面应用，所以需要前端代码来控制路由逻辑，通过浏览器提供的history API可以监听浏览器url的变化，从而控制路由相关操作。dva实例提供了router方法来控制路由，使用的是react-router

		import { Router, Route} from ‘dva/router
		app.router(({history}) => 
			<Router history={history}>
				<Route path=“/” component={HomePage}>
			</Router>
		)
		
4. Model：model是dva中最重要的概念，model非MVC中的M，而是领域模型，用于把数据相关的逻辑聚合到一起，几乎所有的数据，逻辑都在这边进行处理分发。

	·state: 这里的state跟上面的state的概念是一样的，只不过它的优先级比初始化低，基本项目中的state都是在这里定义的。
	·namespace: model的命名空间，同时也是全局state上的属性，只能用字符串，我们在发送action到相应的reducer时，需要用到namespace。
	·Reducer: 以key/value格式定义reducer,用于处理同步操作，唯一可以修改state的地方，由action触发，是一个纯函数。
	·Effect: 用于处理异步操作和业务逻辑，不直接修改state，主要从服务端获取数据，发起action交给reducer处理。
		·put: 用于触发action
		·call 用于调用异步函数，支持promise
		·select 用于从state里获取数据
	·Subscription: 用于订阅一个数据源，根据dispatch相应的action。在app.start()时被执行，数据源可以是当前时间、当前页面url、服务器websoket连接、history路由变化等。

4.Router: 表示路由配置信息，项目中的router.js

	分析下dva数据流图： 首先我们根据url访问相关的Route-Component,在组件中我们通过dispatch发送action到model里面的effect或者直接Reducer，当我们将action发送给effct，基本都是取服务器上面请求数据的，服务器返回数据之后，effect会发送相应的action给reducer,由唯一能改变state的reducer改变state，然后通过connect重新渲染组件。

### connect方法：
connect是一个函数，绑定State到view,connect方法返回的也是一个React组件，通常称为容器组件。因为它是原始UI组件的容器，即在外面包了一层State,connect方法传入的第一个参数是mapStateToProps函数，mapStateToProps函数会返回一个对象，用于建立State到Props的映射关系

	import { connect } from ‘dva’
	
	function mapStateToProps(state) {
		return { todos: state.todos}
	}
	connect(mapStateToProps)(App)

### dispatch
dispatch 是一个函数方法，用来将Action发送给State。dispatch方法是在被connect的Component会自动在props中拥有

### Model对象的属性
· namespace:当前Model的名称。整个应用的state，由多个小的Model的State以namespace为key合成
· state: 该Model当前的状态，数据保存在这里，直接决定了视图层的输出
· reducers: Action处理器，处理同步动作，用来算出最新的State
· effects: Action处理器，处理异步动作

### Reducer
Reducer是Action处理器，用来处理同步操作，可以看做是state的计算器。它的作用是根据Action从上一个state算出当前state.

### Effect
Action处理器，处理异步操作，基于Reduct-saga实现。Effect指的是副作用，根据函数式编程，计算以外的操作都属于Effect，典型的就是I/O操作，数据库读写。

	function *addAfter(action, {put, call}) {
		yield call(delay,1000)
		yield put({type: ‘add’})
	}

### 不吹不黑聊前端框架 -- 尤大大

组件可以是函数： 想象一下整个应用是一个大的函数，函数里面可以调用别的函数，每一个组件是一个函数，一个组件可以调用其他函数，整个一个树状结构。

组件是有分类的： 

	· 纯展示型组件，数据进，DOM出，直观明了
	· 接入型组件，在react场景下的container component，这种组件会跟数据层的service打交道，会包含一些跟服务器或者说数据源打交道的逻辑，container会把数据向下传递给展示型组件
	
	· 交互型组件，典型的例子是对于表单组件的封装和加强，大部分的组件库都是以交互型组件为主，比如说Element ui,特点是有比较复杂的交互逻辑，但是是比较通用的逻辑，强调组件的复用
	· 功能型组件


		
	
	

