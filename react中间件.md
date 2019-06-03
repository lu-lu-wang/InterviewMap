# Redux中间件机制
Redux本身就提供了非常强大的数据流管理功能，但并不影响它唯一强大之处，它还提供了利用中间件来扩展自身功能，以满足用户的额开发需求。Redux middleware提供了一个分类处理action的机会。在middleware中，我们可以检阅每一个流过的action，并挑选出特定类型的action进行相应的操作，以此改变action。
一次redux的数据流，增加了middleware后，我们可以在途中对action进行截获，并进行改变。且由于业务场景的多样性，单纯的修改dispatch和reducer显然不能够满足需求，因此对于redux middleware的设计可以是自由组合的，自由插拔的插件机制，也正由于这个机制，我们在使用middleware时，可以通过串联不同的middleware来满足日常的开发，每一个middleware都可以处理一个相对独立的业务需求且相互串联。不同的中间件之所以可以组合使用，是因为Redux要求所有的中间件必须提供统一的接口，每个中间件的逻辑虽然不同，但只要遵循统一的接口就能和redux以及其它中间件对话了。

# 理解中间件的机制
由于redux提供了applyMiddleware方法来加载middleware，因此我们可以首先看下redux中关于applyMiddleware的源码：

	export default function applyMiddleware(...middlewares){
		return createStore => (...args) {
			// 利用传入的createStore和reducer创建一个store
			const store = createStore(...args)
			let dispatch = () => {
				throw new Error()
			}
			const middleStoreAPI = {
				getState: store.getState,
				disptch: (...args) => dispatch(...args)
			}
			// 让每个middleware带着middlewareAPI这个参数分别执行一遍
			const chain = middlewares.map(middleware=>middleware(middleAPI))
			// 接着compose将chain中的所有匿名函数组装成一个新的函数，即新的dispatch
			disptch = compose(...chain)(store.disptch)
			return {
				...store,
				dispatch
			}
		}
	}
从上述可看出，applyMiddleware这个函数的额核心就在于组合compose，通过将不同的middleware一层一层包裹到原生的dispatch

# 创建中间件
中间件的语法是固定的，它是一个三层的嵌套函数。分别需要传递store、next、action。store层主要是我们需要获取store对象。中间件的链式调用主要通过对next的层层加工来实现，所以要有next层。之所以需要action，是因为我们最终还是个disptch函数,最终还是需要action传参的。

	export default store => next => action => {
		// action前的状态
		// dosomething action
		const returnValue = next(action)
		// action后的操作
		return returnValue
	}

# 加载中间件
当我们想让Redux启用某一个中间件的时候，需要在创建store的时候需要应用那些中间件。

	const middleware = applyMiddleware(Middleware1,Middleware2);
	return createStore(combinedReducer,initData,middleware)

applyMiddleware就是一个用来加载dispatch的工厂，而需要加工什么样的dispatch，则需要我们传入对应的中间件函数。

### redux-saga
redux-saga是一个用于管理应用程序Side Effect（副作用，异步处理函数，如接口请求、访问浏览器缓存）的库，它的目标就是让副作用更容易，执行更高效，测试简单，处理故障变得容易。

一个saga就是应用程序中的一个单独的线程，负责处理副作用。redux-saga是一个redux中间件，意味着这个线程可以通过正常的redux action从主应用程序启动，暂定和取消，它能访问整个redux state也可以dispatch、 redux 、action。

redux-saga是middleware的一种，工作与action和reducer之间，按照原始的redux工作流程，当组件中产生一个action后会直接触发reducer修改state;而实际上，组件中发生action后，在进入reducer之前需要先完成一个异步任务，显然原生是不支持该操作的。
继而saga就在其中充当了处理异步的关键：action被触发后，首先执行异步任务，待完成后会将这个action交给reducer。
### 原理：

saga需要一个全局监听器（watcher saga）,用于监听组件发出的action，将监听到的action转发给对应接收器（worker saga），再由接收器执行具体任务，任务执行完后，再发出另一个action修改state。需注意：watcher saga监听的action和对应worker saga发出的action不能是同一个，否则会造成死循环。
在saga中，全局监听器和接收器都是用generator函数和saga自身的一些辅助函数实现对整个流程的控制
Watch saga是业务触发阶段，worker saga是业务执行阶段

```
Component -> Action1 -> Watcher Saga -> Worker Sage -> Action2 -> Reducer -> Component
```

	