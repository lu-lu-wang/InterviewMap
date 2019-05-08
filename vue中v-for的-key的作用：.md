### vue中v-for的:key的作用：
> 简单来说就是为了提高遍历性能。
> vue或者react中在执行列表渲染时会给每个组件加上key这个属性。
> 要解释key的作用，就不得不介绍下虚拟Diff的算法了
> vue和react都实现了一套虚拟的DOM，使我们可以不直接操作DOM元素，只操作数据便可以重新渲染页面。

## react中使用ref属性
react提供ref属性，主要是对组件真正实例的引用，其实是ReactDOM.render()返回的组件实例；需要区分下，ReactDOM.render()渲染组件时返回的是**组件实例**；而渲染dom元素时，返回的是具体的**dom节点**。

挂载到组件（有状态组件）上的ref，表示对组件实例的引用；而挂载到dom元素上的ref，表示对具体dom元素节点的引用。

ref的值可以设置为字符串（不推荐），也可设置为回调函数。

这个回调函数的执行时机是：

1. 组件被挂载后，回调函数会立即执行，回调函数的参数为该组件的具体实例；
2. 组件卸载或者原来的ref属性本身发生变化的时候，回调函数也会被立即执行，此时回调函数参数为null，以确保内存不泄露。

```
class AudioPlay extends React.Component {
	constructor() {
		super();
		this.timeline = null;
		this.playhead = null;
		this.state = {......}
	}
	...
	render () {
		return (
			......
			<div ref={node=>this.timeline = node}/>
			<div ref={node=>playhead = node}/>
			.....
		)
	}
}
```
此段代码中，ref的回调函数中，参数node为div这个dom，渲染完毕后，这个实例将被保存在组件的this.timeline和this.playhead中，使用的时候可以直接使用this.timeline和this.playhead进行一系列操作。
