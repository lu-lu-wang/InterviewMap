# dvaJs上手

### 初始化
安装dva-cli用于初始化项目：

	npm install dva-cli
	//或者
	npm global add dva-cli

创建项目目录，并进入该目录：

	mkdir my-project
	cd my-project

初始化项目：

	dva init

然后运行npm start 或者yarn start 即可运行项目

### 目录结构
项目初始化后，默认的目录结构如下：

	|- mock
	|- node_mudules
	|- package.json
	|- public
	|- src
		|- asserts
		|- components
		|- models
		|- routes
		|- service
		|- utils
		|- routesr.js
		|- index.js
		|- index.css
	|- .editorconfig
	|- .eslintrc
	|- .gitignore
	|- .roadhogrc.mock.js
	|- .webpackrc

· mock 存放用于mock数据的文件

· public 一般用于存放静态文件，打包时会被直接复制到输出目录（./dist）

· src 文件用于存放项目源代码

	· asserts 用于存放静态资源，打包时会经过webpack处理
	· components 用于存放React组件，一般是该项目公用的无状态组件
	· models 用于存放模型文件
	· routes 用于存放需要connect model的路由组件
	· services 用于存放服务文件，一般是网络请求
	· utils 工具类组件
	· routes.js 路由文件
	· index.js 项目入口文件
	· index.css 一般是公用样式

· .editorconfig

· .eslintrc ESLint 配置文件

· .gitignore Git忽略文件

· .roadhogrc.mock.js Mock配置文件

· .webpackrc 自定义的webpack配置文件，JSON格式，如需要JS格式，可修改.webpackrc.js

## antd按需引入
先安装antd和babel-plugin-import:

	npm install antd babel-plugin-import --save
	# 或者
	yarn add babel-plugin-import

babel-plugin-import 也可以通过-D参数安装到devDependences中，它用于实现按需加载

然后在.webpackrc中添加如下配置：

	{
		‘extraBabelPlugins’: [
			[‘import’,{
				‘libraryName’: ‘antd’,
				‘libraryName’: ‘es’,
				’style’: true
			}]
		]
	}

现在就可以按需引入antd的组件了，如import { Button } from ‘antd’

自定义antd主题

可以在.webpackrc中添加theme字段直接进行主题自定义，但是如果自定义的变量太多，建议将其单独提取出来方便管理。

建议在./src目录下新建名为theme.js的文件，然后在. webpackrc中引入

	{
		‘theme’:’./src/thems.js’
	}
	
	// theme.js示例如下：
	
	export default {
		‘primary-color’: ‘#000’
	}

CSS Modules

使用dva-cli初始化的项目默认已经启用了CSS Modules，如果不想使用CSS Modules，在.webpackrc中添加以下配置即可禁用：

	{
		‘disableCSSModules’: true
	}

### 开发代理

如需要开发过程中代理API接口，在.webpackrc中添加如下配置即可：

	{
		‘proxy’: {
			‘/api’: {
				‘target’: ‘http://your-api-server’,
				‘changeOrigin’: true
			}
		}
	}

#### Mock
如需要mock功能，在.roadhogrc.mock.js中添加配置即可：

	export default {
		‘GET/api/users’: {user:[{username:’admin’}]}
	}

如上配置，当请求api/users时会返回JSON格式的数据

同时也支持自定义函数，如下：

	export default {
		‘POST/api/users’:(req,res)=>{res.end(‘ok’)}
	}

当mock数据太多，可以拆分后放到./mock文件夹中，然后在.roadhogrc.mock.js中引入

### HMR
HMR即热更新模块，在修改代码后不需要刷新整个页面，方便开发调试，可以在.webpackrc中添加配置：

	{
		‘env’: {
			‘development’: {
				‘extraBabelPlugins’: [
					‘dva-hmr’
				]
			}
		}	
	}
	如果无效，可尝试babel-plugin-dva-hmr
	env字段是针对特定环境进行配置，因为HMR只在开发环境下使用，所以配置在development字段即可，运行npm run build时的环境变量为production

### 组件动态加载
dva内置了dynamic方法用于实现组件的动态加载

	import dynamic from ‘dva/dynamic’
	
	const UserPageComponent = dynamic({
		app,
		models: ()=>[
			import(‘./models/users’)
		],
		component: () => import(‘./routes/UserPage’)
	})

### dva-loading
dva-loading是一个用于处理loading状态的dva插件，基于dva的管理effects执行的hook实现，它会在state中添加一个loading字段（该字段可自定义），自动帮你处理网络请求的状态，不需要自己再去写showLoading和hideLoading

在./src/index.js中引入即可

	import creactLoading from ‘dva-loading’
	
	const app = dva()
	app.use(creactLoading(opts))
// opts仅有一个namespace字段，默认为loading

### Model
Model是dva最重要的部分，可以理解为redux、react-redux、redux-saga的封装

通常项目中一个模块对应一个model，一个基本model如下

	import { fetchUsers } from ‘../services/user’
	
	export default {
		namespace: ‘user’,
		state: {
			list: []
		},
		redusers: {
			save(state,action) {
				return {
					...state,
					list: action.data
				}
			}
		},
		effects: {
			*fetch(action,{ put,acll }){
				const users = yield 		                     put(fetchUser,action.data)
			}
		},
		subscriptions: {
			setup({ dispatch, history }){
				return history.listen(({ pathname})=>{
					if(pathname === ‘./user’){
						dispatch({type: ‘fetch’})
					}
				})
			}
		}	
	}

namespace是该model的命名空间，同时也是全局state上的一个属性，只能是字符串，不支持使用 . 创建多层命名空间。

state是状态的初始值

reducer类似于redux中的reducer，它是一个纯函数，用于处理同步操作，是唯一可以修改state的地方，由action触发，它有state和action两个参数。

effects用于处理异步操作，不能直接修改state，由action触发，也可以触发action，它只能是generator函数，并且有action和effects两个参数，第二个参数effects包含put、call、select三个字段，put用于触发action，call用于调用异步处理逻辑，select用于从state中获取数据。

subscriptions用于订阅某些数据源，并根据情况dispatch某些action，格式为（{dispatch,history},done）=> unlistenFunction

如上一个model，监听路由变化，当进入/user页面时，执行effects中的fetch，已从服务端获取用户列表，然后fetch中触发reducers中的save将从服务端获取到的数据保存到state中。

注意，在model中触发这个model中的action时不需要写命名空间，比如在fetch中触发save时时{type: ‘save’}。而从组件中触发action时，就需要带上命名空间了。比如在组件中触发某个fetch时，应该是{type: ‘user/fetch’}

### app
在./src/index.js中可以看到如下代码

	import dva from ‘dva’
	const app = dva()

app就是dva实例,创建实例时dva方法可以传入一些参数，如下：

	· history
	· initialState
	· onError
	· onAction
	· onStateChange
	· onReducer
	· onEffect
	· onHmr
	· extraReducers
	· extraEnhancers

history是给路由用的，默认为hashHistory，如果想要使用browserHistory，需要安装history，然后在./src/index.js引入即可：

	import dva from ‘dva’
	import creactHistory from ‘history/creatBrowserHistory’
	const app = dva({
		history: creatHistory()
	})
initialState是state的初始数据，优先级高于model中的state，默认为{}

其他以on开头的均为钩子函数

### connect
当写完model和组件后，需要将model和组件连接起来，dva提供了connect方法，其实它就是react-redux的connect。

	import React from ‘react’
	import { connect } from ‘dva’
	
	const User = ({dispatch, user})=>{
		return (
			<div></div>
		)
	}
	export default connect(({user})=>{
		return user
	})(User)

connect后的组件除了可以获得dispatch和state，还可以获得location和history

### 错误处理
effects和subscriptions抛出的错误都会经过onError钩子函数，所以可以在onError中进行全局错误处理

	const app = dva({
		onError(err, dispatch){
			console.log(err)
		}
	})

如果需要对某些effects进行特殊的错误处理，可以使用try/catch

### 异步请求
dva集成了isonorphic-fetch用于处理异步请求，并且使用dva-cli初始化项目中，已经在./src/utils/request.js中对fetch进行了简单的封装，可以在这里根据服务端API的数据结构进行统一的错误处理。

# dva框架介绍
dva是基于redux最佳实现实现的framework,简化使用redux和redux-saga时很多繁杂的操作

## 数据流向
数据的改变发生通常是通过用户交互行为或者浏览器行为触发的，当此类行为会改变数据的时候可以通过dispatch发起一个action，如果是同步行为会直接通过Reducer改变State

## Modul
Subscriptions是一种从源获取数据的方法，它的语义是订阅，用于订阅一个数据源，然后根据条件diapatch需要的action。数据源可以是当前的时间、服务器的websocket连接、keyboard输入、geolocation变化、history路由变化等

# Effect
Effect被称为副作用，在我们的应用中，最常见的就是异步操作。它来自于函数编程的概念，之所以叫副作用是因为它使得我们的函数变得不纯，同样的输入不一定获得同样的输出。

dva为了控制副作用的操作，底层引入了reducx-saga做异步流程控制，由于采用了generator的相关概念，所以将异步转成同步写法，从而将effects转化为纯函数。

# Reducer
在dva中，reducers聚合积累的结果是当前的model的state对象。
通过actions中传入的值，与当前reducers中的值进行运算获得新的值（也就是新的state）。需要注意的是Reducer必须是纯函数，所以同样的输入必然得到同样的输出，它们不应该产生任何副作用，并且，每一次的计算都应该使用immutable data,这种特性简单理解就是每次操作都是返回一个全新的数据（独立，纯净），所以热重载和时间旅行这样的功能才能使用。

	reducers: {
		initList(state,action){
			return {
				...state,
				...action.data
			}
		}
	}

# State
State表示Model的状态数据，通常表现为一个js对象（当然它可以是任何值）；操作的时候每次都要当作不可变数据（immutable data）来对待，保证每次都是全新的对象，没有引用关系，这样才能保证State的独立性，便于测试和追踪变化。

# Action
Action是一个普通的js对象，它是改变State的唯一途径。无论是从UI事件、网络回调，还是websocket等数据源所获得的数据，最终都是会通过dispatch函数调用一个action，从而改变对应的数据，action必须带有type属性指明具体的行为，其它字段可以自定义，如果要发起一个action需要用dispatch函数；需要注意的是dispatch是在组件connect Models以后，通过props传入的。dispatching function 是一个用于触发action的函数，action是改变State的唯一途径，但是它只描述了一个行为，而dispatch可以看作是触发这个行为的方式，而Reducer则是描述如何改变数据的，在dva中，connect Model的组件通过props可以访问到dispatch，可以调用model中的Reducer或者effects

# Route Components
在dva中我们通常将其约束为Route Components,因为在dva中我们通常以页面维度来设计container Components,所以在dva中，通常需要connect Model的组件都是Route Components，组织在/routes/目录下，而/components目录下则是纯属组

# connect
通过connect将modul中的元素作为props的方式传递给component


	

	
	
	