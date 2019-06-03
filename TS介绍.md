# TS介绍
[TypeScript](https://ts.xcatliu.com/basics/type-of-object-interfaces)

为了让程序有价值，我们需要能够处理最简单的数据单元：数字，字符串，结构体，布尔值等。ts支持与js几乎相同的数据类型，此外还提供了使用的枚举类型方便我们使用。
> 格式： const item: 类型 = 值


##什么是TypeScript
TypeScript是js的一个超集，主要提供了*类型系统*和*对ES6的支持*。可以编译成纯js。编译出来的js可以运行在任何浏览器上，TS编译工具可以运行在任何服务器和任何系统上。
###TS的优势：
#### TypeScript增加了代码可读性和可维护性
* 类型系统实际上最好的文档，大部分的函数看烈性定义就指导如何使用了。
* 可以在编译阶段就发现大部分错误
* 增强了编辑器和IDE的功能，包括代码不全、接口提示、跳转到定义、重构等

####TypeScript非常包容
* TypeScript是js的超集，.js文件可以直接重命名为.ts即可
* 即使不显示的定义类型，也可自动做出类型推论
* 可以定义从简单到复杂的几乎一切类型
* 即使ts编译报错，也可以生成js文件
* 兼容第三方库，即使第三方库不是用ts写的，也可以编写单独的类型文件提供ts读取

####TypeScript拥有活跃的社区
* 大部分第三方库都提供给ts类型定义文件
* ts拥抱ES6规范，支持部分ESNext草案规范

#### TypeScript缺点
任何事物都有两面性，TypeScript同样：
* 有一定的学习成本，需要理解接口（Interfaces）、泛型（Generics）、类（Classes）、枚举类型（Enums）等前端工程师可能不熟悉的概念，
* 短期可能会增加开发成本，毕竟要写一些类型定义，不过对于一个长期维护的项目来讲TS能减少其维护成本。
* 继承到构件流程需要工作量
* 可能和一些库结合不完美

#### TypeScript只进行静态检查，如果发现有错误，编译时会报错
#### TypeScript编译即使报错，还是会生成编译结果
*若在编译报错时终止js生成，可以在tsconfig.json中配置noEmitOnError即可*

## 原始数据类型
js的类型分为两种：原始数据类型和对象类型，原始数据类型包括：布尔值、数值、字符串、null、undefined、symbol

### 布尔值
布尔值是最基础的数据类型，在TypeScript中，使用boolean定义布尔值类型。

```
let isDone: boolean =  false
```
### 数字
使用*number定义数值类型*和js一样，ts里的所有数字都是浮点数。这些浮点数的类型是number。除了支持十进制和十六进制字面量，ts还支持ECMAScript2015中引入的二进制和八进制字面量。

```
	let decLiteral: number = 6;
```

### 字符串
使用*string定义字符串类型*js程序对的另一项基本操作是处理网页或服务器端的文本数据，像其他语言一样，我们使用string表示文本数据类型。和js一样，可以使用双引号或单引号表示字符串。

```
	let name: string = ‘wanglu’
	// 你还可以使用模板字符串，它可以定义多行文本和内嵌表达式。这种字符串是被反引号包围，并以${expr}这种形式嵌入表达式。
	let name: string = ‘wanglu’
	let age: number = 26
	let info: string = `hello, ${name}`
```
### 空值
js没有空值概念，在TypeScript中可以用*void表示没有任何返回值的函数*：
```
function alertName(): void {
	alert(‘hello’)
}
```
声明一个void类型的变量没有什么用，因为你只能将它赋值为undefined和null：
```
	let unusable: void = undefined
```
### Null和Undefined
在TypeScript中，可以使用null和undefined来定义这两个原始数组类型：（仅支持相互赋值）
```
  let u: undefined = undefined;
  let n: null = null
```
与void区别是，undefined和null是所有类型的子类型。也就是说undefined类型的变量，可以赋值给number类型的变量：
```
let num: number = undefined
```
而void类型的变量不能赋值给number类型的变量：
```
 let u: void
 let num: number = u // 报错void不能分配给number
```
## 任意值
任意值（Any）用来表示允许赋值为任意类型。
### 什么是任意类型
如果是一个普通类型，在赋值过程中改变类型是不被允许的：
```
	let name: string = ’27’
	name = 27 // 会报错number类型不能分配给string
	// 改为any类型，则允许被赋值为任意类型
	let name: any = ’27’
	name = 27
```
### 任意值的属性和方法
在任意值上访问任何属性都是被允许的
```
	let anyThing: any = ‘hello’;
	console.log(anyThing.myName)
```
允许调用任何方法：
```
	let anyThing: any = ’Tom’
	anyThing.setName(‘Jack’)
```
**可以认为，声明一个变量为任意值之后，对它的任何操作，返回的内容都是任意值**
### 未声明类型的变量默认被识别为任意值类型
### 如果变量定义的时候没有赋值，不管之后有没有赋值，都会被推断成any类型而完全不被类型检查
```
let name;
name=’27’
name=27
```
### 联合类型
联合类型表示取值可以为多种类型中的一种。
#### 简单的例子
```
let name: string | number
name = ’27’
name = 27
name = true // 会报错boolean类型不能分配给string|number
```
> 联合类型使用 | 分隔每个类型

###访问联合类型的属性或方法
当typescript不确定一个联合类型的变量到底是那个类型的时候，我们只能访问此联合类型的所有类型里共有的属性或方法：
```
	function getLength(something: string | number): number{
		return something.length; // 会报错length不存在于string|number上（主要是不存在于number上）
	}
```
由于length不是string和number共有的属性，所以会报错。
访问string和number共同属性是没问题的：
```
	function getStirng(something: number| string): string{
		return something.toString()
	}
```

### 对象的类型--接口
在TypeScript中，我们使用接口来定义对象的类型
####什么是接口？
在面向对象语言中，接口是一个很重要的概念，它是对行为的抽象，而具体如何行动需要由类去实现。TypeScript中的接口是一个非常灵活的概念，除了可用于对类的一部分行为进行抽象外，也常用于[对象的形状]进行描述。
### 简单的例子
```
	interface Person {
		name: string;
		age: number
	}
	let tom:Person = {
		name: ’Tom’,
		age: 25
	}
```
上面的例子中，我们定义了一个接口Person，接着定义了一个变量tom,它的类型是Person，这样，我们就约束了tom的类型必须和接口保持一致。
###任意属性
我们希望一个接口允许有任意属性，可以使用：
```
	interface Person {
		name: string,
		age?: number,
		[propName: string]: any
		
	}
	let tom: Person = {
		name: ‘Tom’,
		gender: ‘male’
	}
```
使用[propName: string]定义了任意属性取string类型的值。
需注意： 一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集：
```
	interface Person {
		name: string;
		age?: number;
		[propName: string]: string
	}
	let tom: Person = {
		name: ‘Tom’,
		age: 25,
		gender: ‘male’
	}
```
上述会报错，因为任意属性的值是string,但是可选属性age的值却是number,number不是string的子属性，所以报错。
### 只读属性
有时候我们希望对象中的一些字段只能在创建的时候被赋值，那么可以用readonly定义只读属性：
```
	interface Person {
		readonlu id: number;
		name: string;
		age?: number;
		[propName: string]: string
	}
	let tom: Person = {
		id: 000,
		name: ‘tom’,
		gender: ‘male’
	}
	tom.id = 99
```
上例中，使用readonly定义的属性id初始化后，又被赋值，所以报错。
注意，只读的约束在于第一次给对象赋值的时候，而不是第一次给只读属性赋值的时候：

### 数组
ts和js一样都可以操作数组元素，有两种方式可以定义数组.

🌰
	
	// 第一种: 可以在元素类型后面接上[]，表示由此类型元素组成一个数组
	
	let list: number[] = [1,2,3];
	
	// 第二种：使用数组泛型，Array<元素类型>
	let list: Array<number> = [1,2,3];

### 元祖 Tuple
元祖类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。比如，你可以定义一对值分别为string和number类型的元祖。

	let x: [string, number];
	x = [‘hello’, 26]
	
	// 当访问一个已知索引的元素，会得到正确的类型
	console.log(x[0].substr(1))
	
	//当访问一个越界的元素，会使用联合类型替代：
	x[3] = ‘world’ // 字符串可以赋值给(string | number) 类型

### 枚举
enum类型是对js标准数据类型的一个补充。像C#等其他语言一样，使用枚举类型可以为一组数组赋予友好的名字。

	enum Color {Red, Blue};
	let c: Color = Color.Blue;	
	
	// 默认情况下，从0开始为元素编号，你也可以手动指定成员的数值，例如我们将1开始编号
	enum Color {Red = 1, Blue};
	let c: Color = Color.Blue;
	
	// 枚举类型提供的一个便利是你可以由枚举的值得到它的名字
	enum Color {Red = 1, Blue};
	let colorName: string = Color[2]
	alert(colorName) // 显示Blue,因为上面代码中它的值是2

### Any
有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型，这些值可能来自于动态的内容，比如来自用户输入或者第三方代码库。这种情况下，我们不希望类型检查器对这些值进行检查而是直接让他们通过编译阶段的检查，我们可以使用any类型来标记这些变量。

	let notSure: any = 1
	notSure = ‘haha’
	notSure = true

在对现有代码进行改写的时候，any类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查，你可能认为是Object有相似的作用，就像它在其他语言中那样，但是Object类型的变量只是允许你给它赋任意值。

	当知道部分数据类型时，any类型也是有用的。
	let list: any[] = [1,’haha’,false]
	list[1] = 100

### Void
某种程度上来说，void类型像是与any类型相反，它表示没有任何类型，当一个函数没有返回值时，你通常会见到其返回值类型是void：

	function warnUser(): void {
		alert(‘this is my warning message’);
	}
	声明一个viod类型的变量没有什么大用，因为你只能为它赋予undefined和null:
	let unusable: void = undefined;

### Null和Undefined
ts里，undefined和null两者各自有自己的类型分别叫做undefined和null。和void相似，他们的本身类型用处不大

	let u: undefined = undefined;
	let n: null = null;

默认情况下null和undefined是所有类型的子类型，就是说你可以把null和undefined赋值给number类型的变量。

### Never
never类型是任何类型的子类型，也可以赋值给任何类型，然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。即使any也不可以赋值给never。

	下面一些返回never类型的函数：
	// 返回never的函数必须存在无法到达的终点
	function error(message: string): never {
		throw new Error(message)
	}

### 类型断言
有时候你会遇到这样的情况，你会比ts更了解某个值得详细信息，通常这会发生在你清楚地指导一个实体具有比它现有类型更确切的类型。类型断言好比其他语言里的类型转换，但是不进行特殊的数据检查和解构。它没有运行时的影响，只是在编译阶段起作用。

	类型断言有两种形式，其一‘尖括号’语法：
	let someValue: any = ‘this is a string’;
	let strLength: number = (<string>someValue).length;

## 变量声明
let和const是js里相对较新的变量声明方式。let在很多方面与var相似，可避免js里常见问题。const是对let的一个增强，它能阻止对一个变量的赋值。因为ts是js的超集，所以它本身就支持let和const。

### let声明
块级作用域，当用let声明一个变量，它使用的是词法作用域或块级作用域。不同于var声明变量那样可以在包含它们的函数外访问，块级作用域变量在包含它们的块或for循环之外是不能访问的。

拥有块级作用域的变量的另一个特点是，它们不能再被声明之前读或写。虽然这些变量始终‘存在’于它们的作用域里，但在直到声明它的代码之前的区域都属于暂时性死区。它只是用来说明我们不能在let语句之前访问它们，ts可以告诉我们这些信息。

	a++;  // 在未声明前使用是不合法的
	let a;
	
	// 注意,我们仍然可以在一个拥有块级作用域变量被声明前获取它。只要我们不在变量声明前去调用那个函数，

### 重定义及屏蔽
var可多次声明，只得到一个，let不可多次声明，会报错。并不是说块级作用域不能用函数作用域变量来声明。而是块级作用域变量需要在明显不同的块里声明。

	function f(con, x){
		if(con){
			let x = 10;
			return x
		}
		return x
	}
	f(false,1); // return 1
	f(ture,1); // return 10

在一个嵌套作用域里引入一个新名字的行为称作屏蔽。它是一把双刃剑，他可能会不小心引入新的问题，同时也可能解决一些错误。

### 块级作用域变量的获取
每次进入一个作用域时，它创建了一个变量的环境。就算作用域内代码已经执行完毕，这个环境与其捕捉的变量依然存在。当let声明出现在循环体里时拥有完全不同的行为，不仅是在循环里引入了一个新的变量环境。而是针对每次迭代都会创建一个新作用域。这就是我们在使用立即执行的函数表达式时做的事。

### const声明
const 声明变量的另一种方式。与let声明类似，但被赋值后不能改变。它们拥有与let相同的作用域规则，但是不能对它们重新赋值。也就是说应用的值是不可变的。

#### let VS const
使用最小特权原则，所有变量除了你计划去修改的都应该使用const。基本原则就是如果一个变量不需要对它写入，那么其它使用这些代码的人也不能够写入它们，并且要思考为什么会需要对这些变量重新赋值。使用const也可以让我们更容易的推测数据的流动。

### 解构
#### 解构数组
最简单的解构莫过于数组的解构赋值了：

	let input = [1,2]
	let [first,second] = input
	console.log(first) // 输出 1
	console.log(second) // 输出 2
	
	// 这里创建first、second两个变量。相当于使用了索引
	first = input[0]
	second = input[1]
	
	// 解构作用于已声明的变量会更好
	[first,second] = [second,first]
	
	// 作用于函数参数：
	function f([first,second]:[number,number]){
		console.log(first)
		console.log(second)
	}
	f(input)
	
	你可以在数组里使用...语法创建剩余变量：
	let [first,...rest] = [1,2,3,4];
	console.log(first)   // 输出 1
	console.log(rest)    // 输出 [2,3,4]
	
	或其它元素：
	let [,second,fourth] = [1,2,3,4]

### 对象解构

	let o = {
		a: ‘foo’,
		b: 12
	}
	let {a,b} = o
### 属性重命名
你可以给属性以不同的名字：

	let {a: new1,b:new2} = o
	//令人困惑的是，这些冒号不是指示类型的。如果你想指定它的类型，仍然需要在其后面写上完整的模式。
	let {a,b}:{a:string,b:number} = o;

### 默认值
默认值可以让你在属性为undefined时使用缺省值：

	function keepObject(object:{a: string,b?:number}){
		let {a,b=100} = object
	}
	//现在，即使b为undefined，keepObject函数的变量object的属性a和b都会有值。

### 函数声明
解构也能用于函数声明。

	type C = {a: string, b: number}
	function f({a,b}:C):void {}
	
	// 通常情况下更多的是指默认值，解构默认值有些棘手，首先，要在默认值之前设置其格式
	function f({a,b}={a:’’,b:0}:void){}
	f() // 输出默认值{a:’’,b:0}
	
	// 其次，你需要知道在解构属性上给予一个默认或可选的属性用来替换主初始化列表。要知道C的定义上有一个b可选属性：
	function f({a,b = 0} = {a: ‘’}): void {}
	f({a:’yes’}) // b=0
	f() // b=0
	f({}) // error 
	
	// 要小心使用解构。从例子可看出，就算是最简单的解构表达式也是难以理解的。尤其当存在深层嵌套解构的时候，就算这时没有堆叠在一起重命名，默认值和类型注解，也是令人难以理解的。解构表达式要尽量保持小而简单。你自己也可以直接使用解构将会生成的赋值表达式。

### 展开
展开操作符正与解构相反。它允许你将一个数组展开为另一个数组，或将一个对象展开为另一个对象。

	let first = [1,2]
	let second = [3,4]
	let bothPlus = [0,...first,...second,5]
	//展开操作创建了first和second的一份浅拷贝。它们不会被展开操作符所改变。

## 接口
ts的核心原则之一就是对值所具有的结构进行类型检查。它有时被称作‘鸭式辨型法’或‘结构性子类型化’。在ts里，接口的作用就是为这些类型命名和为你的代码或第三方代码契约。

### 接口初探

	function printLabel(labelObj:{label: string}) {
		conosle.log(labelObj.label)
	}
	let myObj = {size: 10,label:’size 10 object’}
	printLabel(myObj)
	



	
	

	