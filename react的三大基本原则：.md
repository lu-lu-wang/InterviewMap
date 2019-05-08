### react的三大基本原则：
###### 单一的数据源
整个应用的state被存储在一颗object tree中，并且这个object tree只存在于一个唯一的store中。
这让同构应用开发变得非常容易，来自服务端的state可以在无需编写更多代码的情况下被序列化并注入到客户端中，由于是唯一的state tree，调试变得容易，在开发中，你可以把应用的state保存在本地，从而加快开发速度，此外，受益于单一的state tree，以前难以实现的撤销、重做功能变得轻而易举。
###### state是只读的
唯一改变state的方法就是触发action，action是一个用于描述已发生事件的普通对象。
这样确保了视图和网络请求都不能直接修改state。因为所有的修改都被集中化处理，且严格按照一个接一个的顺序执行，因此不用担心race condition的出现。action就是普通对象而已，因此可以被打印，序列化，存储，后期调试等
###### 使用纯函数来执行修改
为了描述action如何修改state tree，你需要编写reducers
Reducer只是一些纯函数，它接收先前的action和state，并返回新的state。刚开始你可以只有一个reducer，随着应用的变大，你可以把他拆成多个小的reducers，分别独立的操作state tree的不同部分，因为reducer只是函数，你可以控制它们被调用的顺序，传入附加数据，甚至编写可复用的reducer来处理一些通用任务，如分页器。
React源码解析
react为什么渲染那么快？
react提供了高效的虚拟DOM以及diff算法，才让react之所以那么快。在内存中的对象，从某种意义上来说，这是非常快速的，如果你用jsx并且阅读过编译之后的代码，请看如下代码：
React.createElement(
	‘div’,
	{
		className: css
	},
	React.createElement(
		‘lable’,
		{className: ‘item-lable’},
		lable
	),
	React.createElenment(
		‘div’,
		{className: ‘item-field’},
		Inputs
	)
)
是的，jsx只是一个语法糖（语法糖的使用其实就是让我们的写的代码更简单，看起来也更容易理解。），看起来我们在书写html，实际上通过工具将其转化成了js的某一个对象上的某一个方法，我们可以将其转化成更简单的对象，我想你应该就可以看明白了。
virtual DOM,当你需要获取某个节点下的数据时，你可以不需要再像以前那样document.getElementById(‘div’).innerHTML,而是直接从内存中来获取数据this.props以及ref对象

### React中的key的作用是什么？

###### Key是React用于追踪那些列表中元素被修改、被添加或者被移除的辅助标识。

	render () {
		return (
			<ul>
				{this.state.todoItems.map(({item,key})=>{
					return <li key={key}> {item} </li>
				})}
			</ul>
		)
	}
	在开发过程中，我们需要保证每个元素的key在其同级元素中具有唯一性。在React Diff算法中React会借助元素的key值来判断该元素是新近创建的还是被移动而来的元素，从而减少不必要的元素渲染。此外，React还需要借助key值来判断元素与本地状态的关联关系，因此我们绝不可忽略视转换函数中key的重要性。

### 调用setState之后发生了什么？
在代码中调用setState函数之后，React会将传入的参数对象与组件当前的状态合并，然后触发所谓的调和过程。经过调和过程，react会以相对高效的方式根据新的状态构建React元素树并且着手重新渲染整个UI界面，在React得到元素树之后，React会自动计算出新的树与老树的节点差异，然后根据差异对界面进行最小化重渲染。在差异计算算法中，React能够相对精确的指导那些位置发生了改变以及应该如何改变，这就保证了按需更新，而不是全部重新渲染。

### react声明周期函数
	· 初始化阶段
		· getDefaultProps:获取实例的默认属性
		· getInitialState: 获取每个实例的初始化状态
		· componentWillMount: 组件即将被装载、渲染到页面上
		· render: 组件在这里生成虚拟DOM节点
		· componentDIdMount: 组件真正在被装载之后
		
	· 运行中状态：
		· componentWillReceiveProps: 组件将要接受属性的时候调用
		· shouldComponentUpdate: 组件接受新属性或者新状态的时候（可以返回fasle，接受数据后不更新，阻止render调用，后面的函数不会被继续执行了）
		· componentWillUpdate:组件即将更新不能修改属性和状态
		· render: 组件重新描绘
		· componentDidUpdate: 组件已经更新
		
	· 销毁阶段：
		· componentWillUnmount: 组件即将被销毁

### shouldComponentUpdate是做什么的，react性能优化是哪个周期函数？
shouldComponentUpdate这个方法用来判断是否需要调用render方法重新描绘dom.因为dom的描绘非常消耗性能，如果我们能在shouldComponentUpdate方法中能够写出更优化的dom diff算法，可以极大提高性能。

### 为什么虚拟dom会提高性能？
虚拟dom相当于js和真实dom中间加了一个缓存，利用dom diff算法避免了没有必要的dom操作，从而提高性能.
用js对象结构表示DOM树的结构；然后用这个树构建一个真正的DOM树，插到文档中当状态变更的时候，重新构造一棵新的对象树，然后用新的树和旧的树进行比较，记录两棵树差异把2所记录的差异应用到步骤1所构建的真正dom树上，视图就更新了。

### react diff原理

	· 把树形结构按照层级分解，只比较同级元素。
	· 给列表结构的每个单元添加唯一的key，方便比较。
	· React只会匹配相同class的component（这里class指的是组件名）
	· 合并操作，调用component的setState方法的时候，React将其标记为一个dirty。到每一个事件循环结束，React检查所有标记dirty的component重新绘制。
	· 选择性子树渲染。开发人员可以重写shouldComponentUpdate提高diff性能。

### React中refs的作用是什么？
refs是React提供给我们的安全访问DOM元素或者某个组件实例的句柄。我们可以为元素添加ref属性然后在回调函数中接受该元素在DOM树中的句柄，该值会作为回调函数的第一个参数返回：

	class CustomForm extends Component {
		handleSubmit = () => {
			console.log(‘input’,this.input.value)
		}
		render () {
			<form>
				<input
					type=‘text’
					ref={(input)=> this.input = input}
				/>
				<button>submit</button>
			</form>
		}
	}
	上述代码中的input域包含了一个ref属性，该属性声明的回调函数会接收input对应的DOM元素，我们将其绑定到this指针以便在其他的类函数中使用。另外值得一提的是，refs并不是类组件的专属，函数式组件同样能利用闭包暂存其值：
	function CustomForm({handleSubmit}){
		let inputElement
		return (
			<form 		onSubmit={()=>handleSubmit(inputElement.value)}>
				<input
				type=‘text’
				ref={(input)=> inputElement = input}
				/>
				<button type=“submit”>Submit</button>
			</form>
		)
	}

### 展示组件和容器组件之间有何不同？

	· 展示组件关系组件看起来是什么。展示专门通过props接受数据和回调，并且几乎不会有自身的状态，但当展示组件拥有自身的状态时，通常也只关心UI状态而不是数据的状态。
	· 容器组件则更关心组件是如何运作的，容器组件会为展示组件或者其他容器组件提供数据和行为，他们会调用Flux action，并将其作为回调提供给展示组件。容器组件经常是有状态的，因为他们是（其他组件的）数据源。

### 类组件（Class component）和函数式组件（Functional component）之间有何不同？

	· State是一种数据结构，用于组件挂载时所需数据的默认值。State可能会随着时间的推移而发生突变，但多数时候是作为用户事件行为的结果。
	· Props则是组件的配置。props由父组件传递给子组件，并且就子组件而言，props是不可变得。组件不能改变自身的props，但是可以把其子组件的props放在一起（统一管理）。Props也不仅仅是数据回调函数也可以通过props传递。


### 类组件和函数式组件之间有何不同？

	· 类组件不仅允许你使用更多额外的功能，如组件自身的状态和声明周期钩子，也能使组件直接访问store并维持状态
	· 当组件仅是接受props，并将组件自身渲染到页面时，该组件就是一个‘无状态组件’，可以使用一个纯函数来创建这样的组件，这种组件也被称为哑组件或展示组件。

### 何为受控组件？
在HTML中，类似(input、textsrea和select)这样的表单元素会维护自身的状态，并基于用户的输入来更新。当用户提交表单时，前面提到的元素的值将随表单一起被发送。但是在React中会有些不同，包含表单元素的组件将会在state中追踪输入的值，并且每次调用回调函数时，如onChange会更新state，重新渲染组件，一个输入表单元素，它的值通过React的这种方式来控制，这样的元素就被称为‘受控元素’

### 何为高阶组件？
高阶组件是一个以组件为参数并返回一个新组件的函数。HOC运行你重用代码、逻辑和引导抽象。最常见的可能是Redux的connect函数。除了简单分享工具库和简单的组合，HOC最好的方式是共享React组件之间的行为。如果你发现你在不同的地方写了大量代码来做同一件事，就应该考虑代码重构为可复用的HOC。

### 为什么建议传递给setState的参数是一个callback而不是一个对象？
因为this.props和this.state的更新可能是异步的，不能依赖他们的值去计算下一个state。

### 除了在构造函数中绑定this，还有其他方法么？
你可以使用属性初始值设定项来正确绑定回调，create-react-app也是默认支持的。在回调中你可以使用箭头函数，但问题是每次组件渲染时都会创建一个新的回调。

### （在构造函数中）调用super(props)的目的是什么？
在super()被调用之前，子类是不能使用this的，在ES2015中，子类必须在constructor中调用super().传递props给super()的原则是便于（在子类中）能在constructor访问this.props。

### 应该在React组件的何处发起Ajax请求？
在React组件中，应该在componentDidMount中发起网络请求，这个方法会在组件第一次‘挂载’（被添加到DOM）时执行，在组件的生命周期中仅会执行一次。更重要的是，你不能保证在组件挂载之前Ajax请求已经完成，如果是这样，也就意味着你将尝试在一个未挂载的组件上调用setState，这将不起作用，在componentDidMount中发起网络请求保证这有一个组件可以更新啦。

### 描述事件在React中的处理方式
React实际上并没有将这些事件附加到子节点本身。React将使用单个事件监听顶层的所有事件。这对于性能是有好处的，这就意味着在更新DOM时，React不需要担心跟踪事件监听器。

### React中有三种构建组件的方式
React.createClass()、ES6 class和无状态函数。

### react组件的划分业务组件技术组件？
	· 根据组件的职责通常把组件分为UI组件和容器组件
	· UI组件负责UI的呈现，容器组件负责管理数据和逻辑。
	· 两者通过react-redux提供connect方法联系起来。

### 简述flux思想
Flux最大的特点就是数据的‘单项流动’


	· 用户访问View
	· view 发出用户的action
	· dispatcher接收action，要求store进行相应更新
	· store更新后，发出change事件
	· view收到‘change’事件后，更新页面

### 了解redux么，说一下redux吧
	· redux 是一个应用数据流框架，主要是解决了组件间状态共享的问题，原理是集中式管理，主要有三个核心方法，action,store,reducer，工作流程是view调用store的dispatch接收action传入store，reducer进行state操作，view通过store提供的getState获取最新的数据，flux也是用来进行数据操作的，有四个组成部分action,dispatch,view,store，工作流程是view发出一个action，派发器接收action，让store进行数据更新，更新完成以后store发出change，view接收change更新视图。Redux和Flux很像。主要区别在于Flux有多个可以改变应用状态的store，在Flux中dispatcher被用来传递数据到注册的回调事件，但是在redux中只能定义一个可更新状态的store，redux把store和dispatcher合并，结构更加简单清晰。
	· 新增state对状态的管理更加明确，通过redux，流程更加规范了，减少手动编码量，提高编码效率，同事缺点是当数据更新时有时候组件不需要，但也要重新绘制，有些影响效率。一般情况下，我们在构建多交互，多数据流的复杂项目应用时才会使用它们。

### reudx的缺点？
	· 一个组件所需要的数据，必须由父组件传过来，而不能像flux中直接从store取。
	· 当一个组件相关的数据更新时，即使父组件不需要用到这个组件，父组件还是会重新render，可能会有效率影响，或者需要写复杂的shuoldComponentUpdate进项判断。




 


		
	

