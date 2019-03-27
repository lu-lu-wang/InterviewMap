# Mobx

提出问题：

Mobx会对什么做出反应？

	‘读取’是对对象属性的间接引用，可以用.或者[]的形式取到这些属性值
	‘追踪函数’是computed表达式、observer组件的render()方法和when、reaction和autorun的第一个入参函数
	‘过程’意味着只追踪那些在函数执行时被读取的observerble
Mobx: 简单、可扩展的状态管理工具。它通过透明的函数响应式编程，使得状态管理变得简单和可扩展。Mobx背后的哲学很简单：任何源自应用状态的东西都应该自动地获得。包括UI、数据序列化、服务器通讯等。

React和Mobx是一对强力组合。React通过提供机制把应用状态转换为可渲染组件树并对其进行渲染。而Mobx提供机制来存储和更新应用状态供React使用。对于应用开发中的常见问题，React和mobx都提供了最优和独特的解决方案。React提供了优化UI渲染的机制，这种机制就是通过使用虚拟DOM来减少昂贵的DOM变化数量，Mobx提供了优化应用状态与React组件同步的机制，这种机制就是使用响应式虚拟依赖状态图标，它只有在真正需要的时候才更新并且永远保持是最新的。

### Observable state(可观察的状态)
MobX为现有的数据结构（如对象，数组和类实例）添加了可观察的功能。通过使用@observable装饰器来给你的类属性添加注解就可以简单完成

🌰

		import { observable } from ‘mobx’;
		
		class Todo{
			id = Math.random();
			@observable title = ‘’;
			@observable finished = false
		}

使用observable很像把对象的属性变成excel的单元格，但和单元格不同的是，这些值不只是原始值，还可以是引用值，比如对象和数组。

### Computed value(计算值)
使用MobX,你可以定义在相关数据发生变化时自动更新的值，通过@computed 装饰器或者利用(extend)Observable时调用的getter/setter函数来进行使用。

🌰

	class TodoList {
		@observable todos = []
		@computed get unfinishedTodoCount() {
			return this.todos.filter(todo => !todo.finished).length;
		}
	}
	当添加一个新的todo或者某个todo的finished属性发生变化时，MobX会确保unfinishedTodoCount自动更新。像这样的计算可以类似于MS Excel这样电子表格程序中的公式，每当只有在需要它们的时候，它们才会自动更新。

## 核心API
### 创建observables
observable(value)

用法：· observable(value)						  · @observable classProperty =  value

Observable值可以是JS基本数据类型、引用类型、普通对象、类实例、数组和映射。

### 实用工具
Provider(mobx-react包)：可以用来使用React的context机制来传递store给子组件。

inject(mobx-react包)：相当于provider的高阶组件，可以用来从React的context中挑选store作为prop传递给目标组件。

toJS：toJS(observableDataStructure,options?).把observable数据结构转换成普通的js对象并忽略计算值。

	options包括：
	· detectCycles: boolean：检查observable数据结构中的循环引用。默认为true
	· exportMapAsObject: boolean: 将ES6 Map作为普通对象导出。默认为true

isArrayLike: 用法：isAction(func),如果给定函数是用action方法包裹的或者是用@acction装饰的话，返回true.

isComputed或isComputedProp: 用法：isComputed(thing)或isComputedProp(thing,property?).如果给定的thing是计算值或者thing指定的property是计算值的话，返回true。

intercept: 用法：intercept(object,property?, interceptor),这个API可以在应用observable的API之前，拦截更改。对于验证、标准化和取消等操作十分有用。

observe: 用法：observe(object,property?,listener,fireImmediately = false)这是一个底层API，用来观察一个单个的observable值。
	




	
