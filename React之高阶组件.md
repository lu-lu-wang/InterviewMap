## React之高阶组件
https://juejin.im/post/5bbb1cd5f265da0af1615718?utm_source=gold_browser_extension

React HOC高阶组件详解： High Order Component（包装组件，后面简称HOC），是React开发中提高组件复用性的高级技巧。HOC并不是React的API，是根据React特性形成的一种开发模式。
HOC具体就是一个接受组件作为参数并返回一个新的组件的方法

	const EnhancedComponent = higherOrderComponent(WrapppedComponent)

在React的第三方生态中，有非常多的使用，比如Redux的connect方法或者React-Router的withrouter方法。

===================   例子来啦  =================

	class CommentList extends React.Component {
		constructor(props) {
			super(props)
			this.handleChange = this.handleChange.bind(this);
			this.state = {
				comments: DataSource.getComments()
			}
		}
		componentDidMount() {
			DataScource.addChangeListener(this.handleChange)
		}
		componentWillUnmount() {
			DataSource.removeChangeListener(this.handelChange)
		}
		this.handleChange() {
			this.setState({
				comments: DataSource.getComments()
			})
		}
		render() {
			return (
				<div>
					{
						this.state.comments.map(comment=>{
							<Comment comment={comment} key={comment.id}/>
						})
					}
				</div>
			)
		}
	}
	// 其过程就是1.在组件渲染后监听DataSource 2.在监听器里面调用setState方法 3. 在unmount里卸载监听器。

若是有多个方法同样走此流程，我们就可以将相似的代码提取并复用，对工程的可维护性和开发效率可以带来明显的提升

	// 使用HOC我们可以提供一个方法，并接收了组件和一些组件间的区别配置作为参数，然后返回一个包装过的组件作为结果。我们可以通过简单的调用该方法来包装组件
	
	const CommentListWithSubscription = withSubscription(
		CommentList,
		(DataSource) => DataSource.getComments()
	)
	const BlogPostWithSubscription = withSubscription(
		BlogPost,
		(DataSource,props) => DataSource,getBlogPost(props.id)
	)
	// 注意，在HOC中我们并没有修改输入的组件，也没有通过继承来扩展组件的目的，一个HOC是一个没有副作用的方法。
	
	//在这个例子中我们把两个组件相似的生命周期方法提取出来，并提供selectData作为参数让输入组件可以选择自己想要的数据。因为withSubscription是个纯粹的方法，所以以后如果有相似的组件，都可以通过该方法进行包装，能够节省非常多的重复代码。

https://zhuanlan.zhihu.com/p/24776678

高阶组件是一种很好的模式，很多react库已经证明了其价值。

### 1.什么是高阶组件？
高阶组件就是一个React组件包裹着另一个React组件

简单的实现：

	function ppHOC(WrappedComponent){
		return class PP extends React.Component {
			render () {
				return 
				<WrappedComponent {...this.props}>
			}
		}
	}

	这里主要是HOC在render方法中返回了一个WrappedComponent类型的React Element。
	<WrappedComponent {...this.props}/>
	// 等价于
	React.createElement(WrappedComponent, this.prosp,null)
	// 在React内部的一致化处理，两者都创建了一个React Element用于渲染。一致化处理可理解为React内部将虚拟DOM同步更新到真是DOM的过程，包括旧虚拟DOM的比较及计算最小DOM操作

### 使用Props Proxy可以做什么？

	· 操作props
	· 通过Refs访问到组件实例
	· 提取state
	· 用其他元素包裹WrappedComponent

### 操作props
你可以读取、添加、编辑、删除传给WrappedComponent的props.当删除或者编辑重要的props时要小心，你可能应该通过命名空间确保高阶组件的props不会破坏WrappedComponent。

	添加新的props，在这个应用中，当前登录的用户可以在WrappedComponent中通过this.props.user访问到。
	function ppHOC(WrappedComponent){
		return class PP extends React.Component {
			render () {
				const newProps = {
					user: currentLoggedInUser
				}
				return <WrappedComponent {...this.props} {...newProps}/>
			}
		}
	}

### 通过Refs访问到组件实例
你可以通过引用ref访问到this（WrappedComponent的实例），但为了得到引用，WrappedComponent还需要一个初始渲染，意味着你需要在HOC的render方法中返回WrappedComponent元素，让React开始它的一致化处理，你可以得到WrappedComponent的实例引用。
		
		function refsHOC(WrappedComponent){
			return class RefsHOC extends React.Component {
				proc(){
					wrappedComponentInstance.method()
				}
				render() {
					const props = Object.assign({},this.props,{ref: this.proc.bind(this)})
					return <WrappedComponent {...props}/>
				}
			}
		}
		Ref的回调函数会在WrappedComponent渲染时执行，你就可以得到WrappedComponent的引用，这可以用来读取/添加实例的props,调用实例的方法。

### 提取state
你可以通过传入props和回调函数把state提取出来，类似于smart component 与dumb component。 	
			






