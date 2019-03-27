# interviewMap

### js

内置类型：
JS分为七种内置类型，七种内置类型又分为两大类： 基本类型和对象（Object）

基本类型有6种：number,string,boolean,null,undefined,symbol

其中JS的数字类型是浮点类型的，没有整型。并且浮点类型基于IEEE 754标准实现，在使用中会遇到某些bug。NaN也属于number类型，并且NaN不等于自身。
对于基本类型来说，如果使用字面量的方式，那么这个变量只是个字面量，只有在必要的时候才会转化为对应的类型

let a = 1        // 这只是字面量，不是number类型
a.toString()    // 使用时候才会转化为对象类型

对象（Object）是引用类型，在使用过程中会遇到浅拷贝和深拷贝的问题。

let a = { name: ‘lu’}
let b = a
b.name = ‘hao’
###### console.log(a)   // 已经被修改 {name: ‘hao’}

Typeof

typeof对于基本类型，除了null都可以显示正确的类型

typeof对于对象，除了函数还会显示object
typeof []   // object
typeof {}   // object
typeof console.log   // funciton

对于null来说，虽然它是基本类型，但是会显示object，这是一个存在很久的Bug
typeof null  // object
PS: 为什么会出现这种情况呢？因为在JS的最初版本中，使用的是32位系统。为了性能考虑使用低位存储了变量的类型信息，000开头代表是对象，然而null表示为全零，所以将它错误的判断为object。虽然现在的内部类型判断代码已经改变了，但是对于这个bug却是一直流传下来。

#### 如果我们想获得一个变量的正确类型，，可以通过Object.prototype.toString.call()。这样我们就可以获得类似[Object Type]的字符串。

### 类型转换

转Boolean

在条件判断时，除了undefined，null，false，NaN, ‘’, 0, -0,其他所有值都转化为true，包括所有对象。

对象转基本类型

对象在转换基本类型时，首先会调用valueOf然后调用toString。并且这两个方法你是可以重写的。

	let a = {
		valueOf() {
			return 0
		}
	}
当然你可以重写Symbol.toPrimitive,该方法在转基本类型时调用优先级最高。

	let a = {
		valueOf () {
			return 0
		},
		toString () {
			return ‘1’
		},
		[Symbol.toPrimitive]() {
			return 2
		}
    }
     1 + a // => 3
    ‘1’ + a // => ’12’

### 四则运算符：

只有当加法运算时，其中一方是字符串类型，就会把另一个也转为字符串类型。其他运算只要其中一方是数字，那么另一方就转为数字。并且假发运算会触发三种类型转换: 将值转换为原始值，转换为数字，转换为字符串。

### ==操作符

比较运算x==y，其中x和y是值，产生true或者false。

### 原型

每一个函数都有prototype属性，除了Function.prototype.bind(),该属性指向原型。每个对象都有__proto__属性，指向了创建该对象的构造函数的原型。其实这个属性指向了[[prototype]],但是[[prototype]]是内部属性，我们并不能访问到，所以使用_proto_来访问。

对象可以通过__proto__来寻找不属于该对象的属性，__proto__将对象连接起来组成了原型链。

### new

1.新生成了一个对象
2.连接到原型
3.绑定this
4.返回新对象
在调用new的过程中会发生以上四件事情，我们可以试着自己实现一个new

	function creat () {
		let obj = new Object()  // 创建一个空对象
		let Con = [].shift.call(arguments) // 链接到原型
		obj.__proto__ = Con.prototype // 链接到原型
		let result = Con.apply(obj, arguments) // 绑定this,执行构造函数
		return typeof reault === ‘object’ ? result : obj // 确保new出来的是个对象
	}

对于实例对象来说，都是通过new产生的，无论是function Foo()还是let a = { b: 1}

对于创建一个对象来说，更推荐使用字面量的方式创建对象，因为你使用new Object()的方式创建对象需要通过作用域一层层找到Object,但是你使用字面量的方式就没有这个问题。

function Foo(){}   // function就是个语法糖内部等同于new  Function()

let a = { b: 1 }   // 这个字面量内部也是使用了new Object()

对于new来说，还需要注意运算符优先级

	function Foo () {
		return this;
	}
	Foo.getName = function () {
		console.log(‘1’)
	};
	Foo.prototype.getName = function () {
		console.log(‘2’)
	}

	new Foo.getName()  // -> 1
	new Foo().getName() // ->2

	new Foo()的优先级大于new Foo,所以对于上述代码来说可以这样划分执行顺序
	new (Foo.getName()) // => 1
	(new Foo()).getName() // =>2
	对于第一个函数来说，先执行了Foo.getName(),所以结果为1；对于后者来说，先执行 new Foo()产生了一个实例，然后通过原型链找到了Foo上的getName函数，所以结果为2.

### instanceof

instanceof可以正确的判断对象的类型，因为内部机制是通过判断对象的原型链中是不是能找到类型的prototype。
实现以下instanceof

	function instanceof(left, right) {
		//获得类型的原型
		let prototype = right.prototype
		// 获得对象的原型
		left = left.__proto__
		// 判断对象的类型是否等于类型的原型
		while (true) {
			if (left === null) {
				return false
			}	
			if (prototype === left) {
				return true
			}
			left = left.__proto__
		}
	}

### this

this是很多人会混淆的概念，但是其实一点都不难😂😂，你只要记住几个规则就可以了。

	function foo () {
		console.log(this.a)
	}
	var a = 2
	foo()
	
	var obj = {
		a: 2,
		foo: foo
	}
	obj.foo()
	
	// 以上两种情况‘this’只依赖与调用函数前的对象，优先级是第二个大于第一个情况
	
	// 以下情况是优先级最高的，‘this’只会绑定在‘c’上，不会被任何方式修改‘this’指向

	var c = new foo()
	c.a = 3
	console.log(c.a)
	
	// 还有种就是利用call，apply, bind改变this,这个优先级仅次于new

以上几种情况明白了，很多代码中的this应该就没什么问题了，下面我们看下箭头函数中的this

	function a() {
		return () => {
			return () => {
				console.log(this)
			}
		}
	}
	console.log(a()()())
	
箭头函数其实是没有this的，这个函数中的this只取决于他外面的第一个不是箭头函数的this。这个例子中，因为调用a符合前面代码中的第一个情况，所以this是window,并且this一旦绑定了上下文，就不会被任何代码改变。

### 执行上下文

当执行JS代码时，会产生三种执行上下文

·全局执行上下文

·函数执行上下文

·eval执行上下文

每个执行上下文中都有三个重要的属性，

·变量对象VO，包含变量、函数声明和函数的形参，该属性只能在全局上下文中访问

·作用域链（JS采用词法作用域，也就是说变量的作用域是在定义时就决定了）

·this

	var a = 10
	function foo(i){
		var b = 20
	}
	foo()
	
	// 对于上述代码，执行栈中有两个上下文：全局上下文和函数foo上下文
	stack = [globalContext,fooContext]
	
	对于全局上下文来说，VO大概是这样
	globalContext.VO === global
	globalContext.VO = {
		a: undefined,
		foo: <Function>
	}
	
	对于函数foo来说，VO不能访问，只能访问到活动对象AO
	fooContext.VO === foo.AO
	fooContext.AO {
		i: undefined,
		b: undefined,
		arguments: <>
	}
	// arguments是函数独有的对象（箭头函数没有）
	// 该对象是一个伪数组，有length属性且可以通过下标访问元素
	// 该对象中的caller属性代表函数本身
	// caller属性代表函数的额调用者

🌰

	b () //call b
	console.log(a) // undefined
	var a = ‘hello’
	function b(){
		console.log(‘call b’)
	}
上述结果是因为函数和变量提升的原因。通常提升的解释是说将声明的代码移动到了顶部，这其实没有什么错误，便于大家理解，但是更为准确的解释： 在生成执行上下文时，会有两个阶段。第一个阶段是创建阶段，JS解释器会找出需要提升的变量和函数，并且给他们提前在内存中开辟空间，函数的话会将整个函数存入内存中，变量只声明并且赋值为undefined,所以在第二阶段，也是代码执行阶段，我们可以直接提前使用。

在提升的过程中，相同的函数会覆盖上一个函数，并且函数优先于变量提升。
🌰

	b() // call b secound
	function b(){
		console.log(‘call b first’)
	}
	function b(){
		console.log(‘call b secound’)
	}
	var b = ‘hell world’
	
	var会产生很多错误，所以在ES6中引入了let。let不能在声明前使用，但是并不是常说的let不会提升，let提升了声明但是没有赋值，因为临时死区导致了不能再声明💰使用。

对于非匿名的立即执行函数需要注意：

	var foo = 1
	(function foo () {
		foo = 100
		console.log(foo)
	})() // => f foo() {foo = 10; console.log(foo)} 

因为当JS解释器在遇到非匿名的立即执行函数时，会创建一个辅助的特定对象，然后将函数名称作为这个对象的属性，因此函数内部才能访问到foo，但是这个值是只读的，所以对它的赋值并不生效，所以打印的结果还是这个函数，并且外部的值也没有发生更改。

### 闭包

闭包定义简单：函数A返回了一个函数B，并且函数B使用了函数A的变量，函数B就被称为闭包。
🌰

		function a () {
			let a = 1
			function b (){
				conosle.log(a)
			}
			return b
		}
		// 你是否会疑惑，为什么函数a已经弹出调用栈了，为什么函数b还可以引用函数a中的变量，因为函数a中的变量这时候是存储在堆上的，现在的JS引擎可以通过逃逸分析分辨出那些变量需要存储在堆上，那些需要存储在栈上。
经典面试题：

🌰

	for(var i=0;i<=5;i++){
		setTimeout(function timer() {
			console.log(i)
		},1000)
	}
	// 首先因为setTimeout是个异步函数，所以会先把循环全部执行完毕，这时候i就是6了，所以会输出一堆6。

解决办法两种，第一种使用闭包

	for(var i=1;i<=5;i++){
		(function(j){
			setTimeout(function timer() {
				console.log(j)
			},1000)
		})(i)
	}

第二种就是使用setTimeout的第三个参数

	for(var i=0;i<=5;i++){
		setTimeout(function timer(j){
			console.log(j)
		},1000,i)
	}
	
第三种使用let定义i

	for (let i=0; i<=5;i++) {
		setTimeout(function timer(){
			console.log(i)
		})
	}
因为对于let来说，他会创建一个块级作用域，相当于

	{
		// 形成块级作用域
		let i = 0
		{
			let ii = i
			setTimeout(function timer() {
				conosle.log(i)
			},1000)
		}
		i++
		{
			let ii = i
		}
		i++
		{
			let ii = i
		}
		...
	}

### 深浅拷贝
	let a = {
		age: 1
	}
	let b=a
	a.age = 2
	console.log(b.age) // 2
	
	从上述例子中我们可以发现，如果给一个变量赋值一个对象，那么两者的值会是同一个引用，其中一方改变，另一方也会相应改变。

我们如何解决上述问题呢？

### 浅拷贝

首先可以通过Object.assign来解决这个问题
🌰

	let a = {
		age: 1
	}
	let b = Object.assign({},a)
	a.age = 2
	conosle.log(b.age) // -> 1值未被修改
	
	当然我们也可以通过展开运算符（...）来解决
	let a = {
		age : 1
	}
	let b = {...a}
	a.age = 2
	console.log(b.age) // -> 1

通常浅拷贝就能解决大部分问题了，但是当我们遇到如下情况就需要使用到深拷贝了

🌰

	let a = {
		age: 1,
		jobs: {
			first: ‘it’
		}
	}
	let b = {...a}
	a.job.first = ‘po’
	console.log(b.jobs.first) // -> po
	浅拷贝只解决了第一层的问题，如果接下去的值中还有对象的话，那么就又回到刚开始的话题了，两者享有相同的引用。要解决这个问题，我们需要深入拷贝。

### 深拷贝
这个问题可用JSON.parse(JSON.stringify(object))来解决

	let a = {
		age: 1,
		jobs: {
			first: ‘it’
		}
	}
	let b = JOSN.parse(JSON.stringify(a))
	a.jobs.first = ‘po’
	conosle.log(b.jobs.first)  // po

但是该方法有局限性的：


·会忽略undefined

·不能序列化函数

·不能解决循环引用的对象
	
	let a = {
    age: undefined,
    jobs: function() {},
    name: 'yck'
	}
	let b = JSON.parse(JSON.stringify(a))
	console.log(b) // {name: "yck"}

你会发现上述情况下，该方法会忽略掉函数和undefined

但是在通常情况下，复杂数据都是可以序列化的，所以这个函数可以解决大部分问题，并且该函数是内置函数中处理深拷贝性能最快的。当然如果你的数据中包含有以上三种情况下，可以使用loadsh的深拷贝函数。

	_.chunk(array, [size = 1])
	
	例子：
	_.chunk([‘a’,’b’,’c’,’d’],2)
	// => [[‘a’,’b’],[‘c’,’d’]]
	
	_.chunk([‘a’,’b’,’c’,’d’],3)
	//=> [[‘a’,’b’,’c’],[‘d’]]

### 模块化

在有Babel的情况下，我们可以直接使用ES6的模块化

栗子

	export function a(){}
	export function b(){}
	export default function(){}
	import {a,b} from ‘./a.js’
	import xxx from ‘./b.js’

#### CommonJS

CommonJS是Node独有的规范，浏览器中使用就需要用到Browserify解析了

	module.export = {
		a: 1
	}
	// 或者
	
	exports.a = 1
	// b.js
	var module = require(‘./a.js’)
	module.a // => log 1

上述代码中，module.exports和exports 很容易混淆，其实现如下代码：

	var module = require(‘./a.js’)
	module.a
	//这里其实就是包装了一层立即执行函数，这样就不会污染全局变量了
	// 重要的是mudule这里，module是Node独有的一个变量
	module.exports = {
		a: 1
	}	
	// 基本实现
	var module = {
		exports: {}  // exports就是个空对象
	}
	// 这个是为什么exports 和module.exports 用法相似的原因
	var exports = module.exports
	var load = function (module) {
		// 导出的东西
		var a = 1
		module.exports = a
		return mudule.exports
	}
再来说说module.exports和exports，用法其实是相似的，但是不能对exports直接赋值，不会有任何效果。

对于CommonJS和ES6中的模块化的两者区别是：

·前者支持动态导入，也就是require(${path}/xx.js),后者目前不支持。

·前者是同步导入，因为用于服务端，文件都在本地，同步导入即使卡住主线程影响也不大。而后者是异步导入，因为用于浏览器，需要下载文件，如果也采用导入会对渲染有很大影响

·前者在导出时都是值拷贝，就算导出的值变了，导入的值也不会变，所以如果想要更新值，必须重新导入一次。但是后者采用实时绑定的方式，导入导出的值都是指向同一个内存地址，所以导入的值会随导出的值变化

·后者会编译成require/exports来执行

#### AMD
AMD是由RequireJS提出的

	// AMD
	define([‘./a’,’./b’],function(a,b){
		a.do()
		b.do()
	})
	define(function(require,exports,module){
		var a = require(‘./a’)
		a.doSomething()
		var b = require(‘./b’)
		b.doSomething()
	})

### 防抖

事件：在滚动事件中需要做个复杂计算或者实现一个按钮的防二次点击操作。（本质：函数去抖就是对于一定时间段的连续的函数调用，只让其执行一次）

这些需求都可以通过函数防抖来实现。尤其是第一个需求，如果在频繁的事件回调中做复杂计算，很有可能导致页面卡顿，不如将多次计算合并为一次计算，只在一个精确点做操作，因为防抖动的轮子很多，这里也不重新自己造轮子了，直接使用underscore的源码来解释防抖动。

	/**
 	* underscore 防抖函数，返回函数连续调用时，空闲时间必须大于或等于 wait，func 才会执行
 	*
 	* @param  {function} func        回调函数
 	* @param  {number}   wait        表示时间窗口的间隔
 	* @param  {boolean}  immediate   设置为ture时，是否立即调用函数
 	* @return {function}             返回客户调用函数
 	*/
 	
	_.debounce = function(func, wait, immediate) {
    var timeout, args, context, timestamp, result;
    var later = function() {
      // 现在和上一次时间戳比较
      var last = _.now() - timestamp;
      // 如果当前间隔时间少于设定时间且大于0就重新设置定时器
      if (last < wait && last >= 0) {
        timeout = setTimeout(later, wait - last);
      } else {
        // 否则的话就是时间到了执行回调函数
        timeout = null;
        if (!immediate) {
          result = func.apply(context, args);
          if (!timeout) context = args = null;
        }
      }
    };
    return function() {
      context = this;
      args = arguments;
      // 获得时间戳
      timestamp = _.now();
      // 如果定时器不存在且立即执行函数
      var callNow = immediate && !timeout;
      // 如果定时器不存在就创建一个
      if (!timeout) timeout = setTimeout(later, wait);
      if (callNow) {
        // 如果需要立即执行函数的话 通过 apply 执行
        result = func.apply(context, args);
        context = args = null;
      }

        return result;
      };
    };

整体函数不难，总结一下

·对于按钮防点击来说的实现：一旦我开始一个定时器，只要我定时器还在，不管你怎么点击都不会执行回调函数，一旦定时器结束并设置为null,就可以再次点击了。

·对于延时执行函数来说的实现：每次调用防抖动函数都会判断本次调用和之前的时间间隔，如果小于需要的时间间隔，就会重新创建一个定时器，并且定时器的延时为设定时间减去之前的时间间隔。一旦时间到了，就会执行相应的回调函数。

### 节流
(https://github.com/hanzichi/underscore-analysis/issues/20)

防抖动和节流本质是不一样的。防抖动是将多次执行变为最后一次执行，节流是将多次执行编程每隔一段时间执行
函数节流的核心是，让一个函数不要执行得太频繁，减少一些过快的调用来节流。（降低触发回调的频率：原生拖拽，需要一直监听mousemove事件，在回调中获取元素当前位置，然后重置dom位置，如果不加以控制，每移动一定像素而触发的回调数量是很惊人的，回调中又伴随dom操作，继而引发浏览器的重排与重绘，性能差的浏览器会直接假死，用户体验很糟糕。）

	/**
	 * underscore 节流函数，返回函数连续调用时，func 执行频率限定为 次 / wait
	 *
	 * @param  {function}   func      回调函数
	 * @param  {number}     wait      表示时间窗口的间隔
	 * @param  {object}     options   如果想忽略开始函数的的调用，传入{leading: false}。
	 *                                如果想忽略结尾函数的调用，传入{trailing: false}
	 *                                两者不能共存，否则函数不能执行
	 * @return {function}             返回客户调用函数   
	 */
	 
	 
	 _.throttle = function(func, wait, options) {
	    var context, args, result;
	    var timeout = null;
	    // 之前的时间戳
	    var previous = 0;
	    // 如果 options 没传则设为空对象
	    if (!options) options = {};
	    // 定时器回调函数
	    var later = function() {
	      // 如果设置了 leading，就将 previous 设为 0
	      // 用于下面函数的第一个 if 判断
	      previous = options.leading === false ? 0 : _.now();
	      // 置空一是为了防止内存泄漏，二是为了下面的定时器判断
	      timeout = null;
	      result = func.apply(context, args);
	      if (!timeout) context = args = null;
	    };
	    return function() {
	      // 获得当前时间戳
	      var now = _.now();
	      // 首次进入前者肯定为 true
		  // 如果需要第一次不执行函数
		  // 就将上次时间戳设为当前的
	      // 这样在接下来计算 remaining 的值时会大于0
	      if (!previous && options.leading === false) previous = now;
	      // 计算剩余时间
	      var remaining = wait - (now - previous);
      context = this;
      args = arguments;
      // 如果当前调用已经大于上次调用时间 + wait
      // 或者用户手动调了时间
 	  // 如果设置了 trailing，只会进入这个条件
	  // 如果没有设置 leading，那么第一次会进入这个条件
	  // 还有一点，你可能会觉得开启了定时器那么应该不会进入这个 if 条件了
	  // 其实还是会进入的，因为定时器的延时
	  // 并不是准确的时间，很可能你设置了2秒
	  // 但是他需要2.2秒才触发，这时候就会进入这个条件
      if (remaining <= 0 || remaining > wait) {
        // 如果存在定时器就清理掉否则会调用二次回调
        if (timeout) {
          clearTimeout(timeout);
          timeout = null;
        }
        previous = now;
        result = func.apply(context, args);
        if (!timeout) context = args = null;
      } else if (!timeout && options.trailing !== false) {
        // 判断是否设置了定时器和 trailing
	    // 没有的话就开启一个定时器
        // 并且不能不能同时设置 leading 和 trailing
        timeout = setTimeout(later, remaining);
      		}
      		return result;
    	};
  	};

### 继承

在ES5中，我们可以使用如下方式解决继承的问题

	function Super() {}
	Super.prototype.getNumber = function() {
		return 1
	}
	function Sub() {}
	let s = new Sub()
	Sub.prototype = Object.create(Super.prototype,{
		constructor: {
			value: Sub,
			enumerable: false,
			writable: true,
			configurable: true
		}
	})
	以上继承实现思路就是将子类的原型设置为父类的原型

在ES6中，我们可以通过class语法轻松解决这个问题

	class MyDate extends Date {
		test(){
			return this.getTime()
		}
	}
	let myDate = new MyDate()
	myDate.test()

但是ES6不是所有浏览器都兼容，所以我们需要使用bable来编译这段代码。

因为在JS底层有限制，如果不是由Date构造出来的实例的话，是不能调用Date里的函数的，所以这也说明：ES6中的class继承与ES5中的一般继承写法是不同的。

既然底层限制了实例必须由Date构造出来，那么我们可以改变下思路实现继承

	function MyData(){}
	MyData.prototype.test = function () {
		return this.getTime()
	}
	let d = new Date()
	Object.setPrototypeOf(d, MyDate.prototype)
	Object.setPrototype(MyData.prototype, Date.prototype)

以上集成实现思路：先创建父类实例 =>改变实例原先的 _proto__ 转而连接到子类的prototype => 子类的prototype的__proto__ 改为父类的prototype。

### call,apply,bind区别

首先说下前两者的区别。

call和apply都是为了改变this的指向，作用都是相同的，只是传参的方式不同。
除了第一个参数外，call可以接收一个参数列表，apply只接收一个参数数组。

🌰

	let a = {
		value: 1
	}
	function getValue(name, age){
		console.log(name)
		console.log(age)
		console.log(this.value)
	}
	getValue.call(a, ‘ll’,’25’)
	getValue.apply(a,[‘ll’,’25’])

模拟实现call和apply

可以从以下几点来考虑和实现

·不传入第一个参数，那么默认为window

·改变了this指向，让新的对象可以执行该函数，那么思路是否可以变成给新的对象添加一个函数，然后在执行完以后删除？

	Function.prototype.myCall = function (context){
		var context = context || window
		// 给context添加一个属性
		// getValue.call(a,’ll’,’25’) => a.fn = getValue
		context.fu = this
		// 将context 后面的参数取出来
		var args = [...arguments].slice(1)
		// getValue.call(a,’ll’,’25’) => a.fn(‘ll’,’24’)
		var result = context.fn(...args)
		// 删除 fn
		delete context.fn
		return result
	}

以上就是call的思路，apply的实现类似

	Function.prototype.myApply = function(context){
		var context = context || window
		context.fn = this
		var result
		//需要判断是否存储第二个参数
		// 如果存在，就将第二个参数展开
		if(arguments[1]){
			result = context.fn(...arguments[1])
		} else {
			result = context.fn()
		}
		delete context.fn
		return result
	}

bind和其他两个方法也是一致的，只是该方法会返回一个函数，并且我们可以通过bind实现科里化。

模拟实现bind：

	Function.prototype.myBind = function(context) {
		if (typeof this !== ‘function’) {
			throw new TypeError(‘Error’)
		}
		var _this = this
		var args = [...arguments].slice(1)
		// 返回一个函数
		return function F() {
			// 因为返回了一个函数，我们可以用new F(),所以需要判断
			if (this instanceof F){
				return new _this(...args,...arguments)
			}
			return _this.apply(context,args.concat(...arguments))
		}
	}

### Promise实现

Promise是ES6新增的语法，解决了回调地狱的问题😏😏

可以吧Promise看成一个状态机。初始值是pending状态，可以通过函数resolve和reject，将状态转变为resolved获得rejected状态，状态一旦改变就不能再变化。

then函数会返回一个Promise实例，并且该返回值是一个新的实例而不是之前的实例。因为Promise规范除了pending状态，其他状态是不可改变的，如果返回的是一个相同实例的话，多个then调用就失去意义了。

对于then来说，本质上可以看做flatMap

	// 三种状态
	const PENDING = ‘pending’
	const RESOLVED = ‘resolved’
	const REJECT = ‘reject’
	//Promise 接收一个函数参数，该函数会立即执行
	function MyPromise(fn){
		let _this = this
		_this.currentState = PENDING
		_this.vlaue = undefined
		// 用于保存then中的回调，只有当promise状态为pending时才会缓存，并且每个实例至多缓存一个
		_this.resolvedCallbacks = []
		_this.rejectCallbacks = []
		_this.resolve = function (value) {
			if(value instanceof MyPromise){
				// 如果value是个Promise，递归执行
				return value.then(_this.resolve,_this.reject)
			}
			setTimeout(()=>{ // 异步执行，保证执行顺序
				if (_this.currentState === PENDING) {
					_this.currentState = RESOLVED
					_this.value = value
					_this.resolvedCallbacks.forEach(cb => cb())
				}
			})
		}
		_this.reject = function (reason) {
			setTimeout(()=>{ // 异步执行，保证执行顺序
				if (_this.currentState ===PENDING){
					_this.currentState = REJECTED
					_this.value = reason
					_this.rejectedCallbacks.forEach(cb => cb())
				}
			})
		}
	}
	......

### Generator实现
Generator是ES6中新增的语法，和Promise一样，都可以用来异步编程

	// 使用*表示这是一个Generator函数，内部可以通过yield暂停代码，通过next恢复执行
	
	function* test() {
		let a = 1 + 2
		yield 2
		yield 3
	}
	let b = test()
	console.log(b.next()) // {value:2,done: false}
	console.log(b.next()) // {value:3,done: false}
	console.log(b.next()) // {value:undefined,done: true}
	从上述代码可以发现，加上*的函数执行后拥有了next函数，也就是说函数执行后返回了一个对象，每次调用next函数可以继续执行被暂停的代码。generator的基本实现如下：
	// cb也就是编译过的test 函数
	function generator(cb){
		return (function() {
			var object = {
				next: 0,
				stop: function() {}
			}
			return {
				next: function() {
					var ret = cb(object)
					if(ret === undefined) 
					return {
						value: undefined,
						done: true
					}
					return {
						value: ret,
						done: false
					}
				}
			}
		})()
	}

// 如果你要使用bable编译后可以发现test函数变成了这样

	function test() {
		var a;
		return generator(function(_context) {
			while(1) {
				switch ((_context.prev = _context.next)) {
					// 可以发现通过yield将代码分割几块
					// 每次执行 next 函数就执行一块代码
					// 并且表明下次需要执行那块代码
					case 0:
					  a = 1 + 2
					  _context.next = 4
					  return 2
					case 4:
					  _context.next = 6
					  return 3
					 // 执行完毕
					 case 6:
					 case ‘end’:
					   return _context.stop()
				}
			}
		})
	}

### Map、FlagMap 和 Reduce

	Map 作用是生成一个新数组，遍历原数组，将每个元素拿出来做一些变换后append到新的数组
	[1,2,3].map(v => v + 1) // ->[2, 3, 4]
	Map有三个参数，分别是当前索引元素，索引，原数组
	[‘1’, ‘2’, ’3’].map(parseInt)
	// parseInt(‘1’, 0) -> 1
	// parseInt(‘2’, 1) -> NaN
	// parseInt(‘3’, 2) -> NaN

FlapMap和map的作用几乎是相同的，但是对于多维数组来说，会将原数组降维。可以将FlagMap看成是map + flatten,目前该函数在浏览器中还不支持

	[1, [2], 3].flagMap(v => v+1)
	// -> [2, 3, 4]

如果还想将一个多维数组彻底降维，可以这样实现

	const flattenDeep = arr => Array.isArray(arr)
	? arr.reduce((a,b) =>
	[...a, ...flattenDeep(b)], [])
	: [arr]
	
	flattenDeep([1, [ [2], [3, [4] ], 5 ]])
	Reduce作用是数组中的值组合起来，最终得到一个值
	function a() {
		console.log(1)
	}
	function b() {
		console.log(2)
	}
	[a, b].reduce((a.b) => a(b())) // -> 2 1

### async和await
一个函数如果加上async,那么该函数就会返回一个Promise

	async function test() {
		return ‘1’
	}
	console.log(test()) // ->Promise {<resolved>: ‘1’}

可以把async看成将函数返回值使用Promise.resolve()包裹了下。


await只能在async函数中使用

	function sleep() {
		return new Promise(resolve => {
			setTimeout(()=>{
				console.log(‘finish’)
				resolve(‘sleep’)
			},2000)
		})
	}
	async function test() {
		let value = await sleep()
		console.log(‘object’)
	}
	test()
	上面的代码会先打印finish然后打印object，因为await会等待sleep函数resolve,所以即便后面是同步代码，也不会先去执行同步代码再来执行异步代码。
	async和await相比直接使用promise来说，优势在于处理then的调用链，能够更清晰准确的写出代码，缺点在于滥用await可能会导致性能问题，因为await会阻塞代码，也许之后的异步代码并不依赖前者，但仍需要等待前者完成，导致代码失去了并发性。
	来看一个使用await的代码
	var a = 0
	var b = async()=>{
		a = a+ await 10
		console.log(‘2’, a) //‘2’ 10
		a = (await 10) + a
		console.log(‘3’, a) ‘3’ 20
	}
	b()
	a++
	console.log(‘1’, a) // ->’1’ 1
	原理：
	1.首先函数b先执行，在执行到await 10之前变量a还是0，因为在await内部实现了generators，generator会保留堆栈中的东西，所以这时候a=0被保存了下来
	2.因为await是异步操作，所以会先执行console.log(‘1’,a)
	3.这个时候同步代码执行完毕，开始执行异步代码，将保留下来的值拿出来使用，这时候a=10
	4.然后后面就是常规代码了

### Proxy
	
	
	

	



	
	







	


	

 		  



	











