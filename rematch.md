### 为什么我们选择Rematch?
之前我们的项目redux-saga、react-router、react，是将所有的数据放在redux中，异步处理数据使用redux-saga。在处理从服务器一步获取数据的时候，我们仅仅是想要从服务端获取一个数据然后保存在store中，为什么我们要引入redux-saga，并且引入后写法很是繁琐，之后了解到##rematch是一个很好的js状态容器##，心里默念终于可以脱离redux了。 

首先我们在官网中了解到，Rematch是没有boilplate的Redux最佳实践，没有多余的action types,aciton creators,switch语句或者thunks。

## 总结：
##### 	1.rematch的优点
		1）省略了action types: 不必再多次写字符串，使用model/method代替
		2）省略了action creators: 直接调用方法，不必再生产action type,使用dispatch.model.method代替
		3）省略了switch语句: 调用model.method方法，不必判断action type
		4）集中书写状态，同步和异步方法: 在一个model中使用state，reducers,effects来写状态，同步和异步方法。

##### 2.rematch的model
	model中直接写state，reducers,effects十分集中方便
	export const count = {
			state: 0,
			reducers: {
				// 修改state的函数
			},
			effects:(dispatch)=>({
				// 此处写异步函数
			})
	}
##### 	3.rematch的dispatch
	dispatch可以直接调用同步和异步方法，不必再发送action
	this.props.changeInfo({
		type: ‘state/changeInfo’,   // 之前写法
		payload: params
	})
	dispatch.count.changeInfo(params)  // rematch写法
##### 4.rematch的状态分发
	依然使用redux


rematch是redux的基础上再次封装后的成果，在rematch中我们不用声明action类型、action创建函数、thunks、store配置、mapDispatchToProps、saga，其写法上和vuex基本一样。

	一个简单的demo：	
	|- index.html
	|- index.js     # 项目的入口
	|- store
		|- index.js     # 引入modules的各个模块，初始化store
		|- modules
			|- count.js   # count模块
			|- info.js	# info模块

	1. store/index.js，初始化一个store实例

		import {init} from ‘@rematch/core’
		import count from ‘./modules/count’
		import info from ‘./modules/info’
		const store = init ({
			models: {
				count,info
			}
		})
		export default store;
		// rematch提供init()方法，返回一个store实例，初始化store的时候rematch支持多个模块，在中、大型项目中可以根据业务需求，拆分成多个模块，这样项目结果就变得清晰了。

	2.store/modules/count.js，编写count模块业务代码
	
		const count = {
			state: {
				num: 1,
				type: 2
			},
			reducers: {
				increment(state,num1,num2){
					...state,
					num: num1
				}
			}
		},
		effects: {
			async incrementAsync(num1,rootState,num2){
				await new Promise(resolve=>setTimeout(resolve,2000));
				this.increment(num1)
			}
		}
		
	export default count;
	// 事实上我们的count模块就是一个对象，该对象包括三个属性：state、reducers、effects
	state: 存放模块状态的地方
	reudcers：改变store状态的地方，每个reducers函数都会返回一个对象作为最新的state。reducers中的函数必须为同步函数，如果要异步处理数据需要在effects中处理。注意：只能通过reducers函数中返回一个新对象来改变模块中state的值，直接通过修改state的方式是不能改变模块的state的值。
	effects：处理异步数据的地方，比如：异步从服务端获取数据。注意：在effects中是不能修改模块state，需要在异步处理完数据后调用reducers中的函数修改模块的state。
	rematch的state、reducers、effects和vuex中的state、mutation、action用法相似，在vuex中mutation修改model的state的值，action进行异步处理数据。
	
	3.在组件中获取state和修改state的值
	
	有两种方法可以获取state和修改state：
	1）使用redux的高阶组件connect将state、reducers、effects绑定到组件props上。
	2）使用rematch提供的dispatch和getState.
	（1）使用redux的高阶组件connect
		使用redux提供的高阶组件connect给App组件注册countState和countDispatch属性，其中countState对应的是count的state属性，countDispatch对应的是count模块的reducers和effects，在组件中使用this.props.countState和this.props.countDispatch就可以访问到count模块提供的state和reducers、effects了。
		import React, { Component} from ‘react’
		import { connect }	from ‘react-redux’
		class App extends Component {
			handleCilck = () => {
				const { countDispatch } = this.props;
				countDispatch.increment(10)
			}
			render () {
				const { countState } = this.props;
				return (
					<div onClick={this.handleClick}>{countState.num}</div>
				)
			}
		}
		const mapStateToProps = (state) => ({
			countState: state.count
		})
		const mapDispatchToProps=(dispatch)=>({
			countDispatch: dispatch.count
		})
		
		export default connect(mapStateToPorps,mapDispatchToProps)(App)

		(2)dispatch和getState
		getState: rematch提供的getState方法返回整个store的state对象，如果要单独访问count模块的state，只需要getState().count即可。
		disptch: rematch提供的dispatch可以直接调用整个store中定义的reducers和effects。dispatch.count.increment(10)，其中count为store中的一个model，increment方法为count模块中提供的一个reducers。调用effects的方法和调用reducers方法一致。
		import React, { Component } from ‘react’
		import { disptch, getState } from ‘@/rematch/core’
		class App extends Component {
			handleClick =()=>{
				console.log(getState().count)
				dispatch.count.increment(10)
			}
			render() {
				let countState = getState().count;
				console.log(countState);
				return (
					<div onClick={this.handleClick}>{countState.num}</div>
				)
			}
		}
	
	4. 在一个model的reducers中的函数触发另一个model的reducers中的函数
	场景：A函数和B函数里有大量相似的代码，这个时候我们一般做法都是将A、B函数公共部分提取出来成一个公共函数，这样我们就不需要在A函数和B函数中写大量相似的代码。加入在Reducers中，我们将A函数和B函数提取的公共函数C放在公共模块info的reducers中，A函数就是在count模块的reducers中，在这种情况下我们就需要在公共模块info的函数执行完后调用count模块的A函数。
	
		