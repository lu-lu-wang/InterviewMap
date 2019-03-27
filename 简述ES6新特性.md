# 简述ES6新特性
1. 变量生命：let、const分别表示变量和常量，都是块级作用域

2. 模板字符串：将变量和字符串连接起来。`hello ${myName}`

3. 默认参数：function fn(a, b=‘1’) { console.log(b) } 

4. 箭头函数：函数的额快捷写法，不需要写function关键字创建函数，可以省略return关键字，同时箭头函数还会继承当前上下文的this关键字。

		[1,2,3].map((item)=>{item})
		// 等同于
		[1,2,3].map(function(item){
			return item
		}).bind(this)

5. 模块的import和export：import用于引入模块，export用于导出模块

		import React from ‘react’
		export default App
		export default class App extends React.Component{}

6. ES6对象和数组：结构赋值，结构赋值让我们从Object或者Array中取出部分数据存为变量

		const user = {name:’wanglu’, age: ’25’}
		const { name,age } = user
		// 数组
		const arr = [ 1,2 ]
		const [ foo, bar ] = arr
		console.log(foo) // 1

7. 对象字面量的改进：这是结构的反向操作，用于重组一个object

		const name = ‘lu’
		const age = ’25’
		const user = { name, age }

8. 展开运算符：... 用于组装数组、获取数组的部分项、收集函数参数为数组、代替apply

		// 组装数组
		const todos = [‘lalala’]
		[...todos,yoyo,haha] // [lalala,yoyo,haha]
		// 获取部分项
		const arr = [a,b,c]
		const [first,...rest] = arr
		rest // [b,c]
		// 收集函数参数为数组
		function directions(first, ...rest){
			console.log(rest)
		}
		directions(‘a’,’b’,’c’) // [‘b’,’c’]
		// 代替apply
		function foo(a,b,c){
			const args = [1,2,3]
		}
		// 等同于
		foo.apply(null,args)
		foo(...args)
		// 对于Object而言，用于组合新的Object
		const foo = {
			a: 1,
			b: 2
		}
		const bar = {
			b: 3,
			c: 2
		}
		const d = 4
		const ret = { ...foo, ...bar, d } // { a:1, b:3, c:2, d:4 }
		此外，在JSX中扩展运算符还可用于扩展props

9. Promise：Promise用于更加优雅的处理异步处理

		fetch(‘/api/todos’)
		.then(res => res.json())
		.then(data => ({data}))
		.catch(err => ({err}))
		
		// 定义Promise
		const delay = timeout => {
			return new Promise(resolve=>{
				setTimeout(resolve, timeout)
			})
		}
		delay(1000).then(_=>{
			console.log(‘executed’)
		})

10. Generator: dva的effects是通过generator组织的，Generator返回的是迭代器，通过yield关键字实现暂停功能。这是一个典型的dva effect，通过yield把异步逻辑通过同步的方式组织起来。

11. React Component: react Component有3种定义方式，分别是React.createClass,class和StatelLess Function Component,推荐使用后者，保持简洁和无状态。这是函数，不是Object，没有this作用域，是纯函数。

		function App(props){
			function handleClick(){
				props.dispatch({
					type: ‘app/create’
				})
			}
		}
		//等同于
		class App extends React.Component {
			handleClick(){
				this.props.disptch({type: ‘app/create’})
			}
			render(){
				return (
					<div onClick={this.handleClick.bind(this)}>${this.props.name}</div>
				)
			}
		}

12. JSX： Component嵌套，类似HTML,JSX里可以给组件添加子组件。className：是class保留词，所以添加样式时，需用className代替class。js表达式：js表达式需用{}括起来，会执行并返回结果。
13. props: 数据处理在React中是非常重要的概念之一，分别可以通过props、state、context来处理数据。而在dva应用里，你只需要关注props即可。propTypes:js是弱类型语言，所以尽量声明propsTypes对props进行校验，以减少不必要的问题。内置的propType有： PropTypes.array、PropTypes.bool、PropTypes.func、PropTypes.number、PropTypes.object、PropTypes.string
14. Reducer：reducer是一个函数，接受state和action,返回老的或新的state。即(state,action) => state
15. 嵌套数据的增删改：建议最多一层嵌套，以保持state的扁平化，深层嵌套会让reducer很难维护。
16. Effects: put,用于触发action，call用于调用异步逻辑，支持promise，select，用于从state中获取数据
17. 错误处理：全局错误处理，在dva里，effects和subscriptions的抛错全部会走onError hook，所以可以在onError里统一处理错误

		const app = dva({
			onError(e,dispatch) {
				console.log(e.message)
			}
		})
		// 然后在effcts里的抛错和reject的promise就会被捕获到了
		本地错误处理： 如果要对某些effects的错误进行特殊的处理，需要在effct内部加try catch
		app.model({
			effects: {
				*addRemote(){
					try {
					// your code here
					} catch(e){
						console.log(e.message)
					}
				}
			}
		})

18. 异步请求： 异步请求基于whatwg-fetch。有GET和POST，有统一错误处理。
19. Subscription： 订阅一个数据源，然后根据需要dispatch相应的action。数据源可以是当前的时间，服务器的websocket连接，keyboard输入，geolocation变化，history路由变化等。格式为： ({ dispatch, history })=> unsubscribe.
20. 异步数据初始化： 当用户进入/users页面时，触发action users/fetch加载用户数据。

		app.model({
			subscriptions: {
				setuo({ dispatch, history }){
					history.listen(({pathname})=>{
						if(pathname === ‘/users’){
							dispatch({
							  type: ‘users/fetch’
							})
						}
					})
				}
			}
		})

21. Router: Router Components通过connect绑定数据

		import { connect } from ‘dva’
		function App() {}
		function mapStateToProps(state, ownPage){
			return {
				users: state.users
			}
		}
		export default connect(mapStateToProps)(App)
		
22. 基于action进行页面跳转

		import { routerRedux } from ‘dva/router’
		
		yield put(routerRedux.push(‘./logout’))
		
		dispatch(routerRedux.push(‘./logout’))
		
		routerRedux.push({
			pathname: ‘/logout’,
			query: {
				page: 2
			}
		})
23. dva配置: Redux Middleware,比如要添加redux-logger中间件：

		import createLogger from ‘redux-logger’
		const app = dva({
			onAction: createLogger()
		})

	
	
	

	

	
