# TS介绍
为了让程序有价值，我们需要能够处理最简单的数据单元：数字，字符串，结构体，布尔值等。ts支持与js几乎相同的数据类型，此外还提供了使用的枚举类型方便我们使用。

## 基础类型

### 布尔值
最基本的数据类型就是简单的true/false，在js和ts里叫做boolean。

🌰

	let isDone: boolean = false

### 数字
和js一样，ts里的所有数字都是浮点数。这些浮点数的类型是number。除了支持十进制和十六进制字面量，ts还支持ECMAScript2015中引入的二进制和八进制字面量。

🌰

	let decLiteral: number = 6;

### 字符串
js程序对的另一项基本操作是处理网页或服务器端的文本数据，像其他语言一样，我们使用string表示文本数据类型。和js一样，可以使用双引号或单引号表示字符串。

🌰

	let name: string = ‘wanglu’
	// 你还可以使用模板字符串，它可以定义多行文本和内嵌表达式。这种字符串是被反引号包围，并以${expr}这种形式嵌入表达式。
	let name: string = ‘wanglu’
	let age: number = 26
	let info: string = `hello, ${name}`

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
	



	
	

	