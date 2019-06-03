#Webpack: 项目配置
[webpack](https://webpack.docschina.org)

[配置 npm start](https://www.jianshu.com/p/00340a8cc943)

相关插件：

浏览器自动打开网页需webpack配置：
[webpack-dev-server](https://webpack.js.org/configuration/dev-server/#devserver-open)

1. webpack

* 基础：为什么要用webpack、安装、基础使用、bundle文件有什么
* Loaders: Bable、处理CSS文件（整合到JS中、整合所有CSS到单独文件、CSS Modules）、处理图片（将小图转为base64格式、PublishPath）

##Webpack是什么？
自模块化后，我们可将一大坨代码分离到各个模块中。打包时每个JS文件都需要从服务器去拿，由此会导致加载速度变慢。Webpack就是将所有的小文件打包成一个或多个大文件。

### 安装
```
mkdir webpack-demo
cd webpack-demo
npm init
npm install --save-dev webpack

```
![安装后目录](https://camo.githubusercontent.com/57dc2113cc60efd119d26f5f641bfecc440742f6/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f31352f3563383864363061363939343235323636303964623633633739313964646166)

在js文件中写入代码

```
// sum.js
module.export = function(a,b){
	return a + b
}
// index.js
var sum = require(./sum)
console.log(sum(1,2))

// index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Document</title>
</head>
<body>
    <div id="app"></div>
    <script src="./build/bundle.js"></script>
</body>
</html>
```
现在我们开始配置简单webpack，首先创建webpack.config.js然后写入：

```
// 自带的库
const path = require(‘path’)
module.exports = {
	entry: ‘./app/index.js’, // 入口文件
	output: {
		path: path.resolve(__dirname, ‘build’)//要使用绝对地址，输出文件夹
		filename: ‘bundle.js’ // 打包后输出文件名
	}
}
```
现在就可以使用webpack了，输入命令

```
node_modules/.bin/webpack
```
![打包截图](https://camo.githubusercontent.com/a59136206f7123ab3976015ee9f97ab155f8a40b/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f31352f6130633237363762383664306233346633626636363232353337376434363633)
因为module.exports浏览器是不支持的,所以webpack将代码改成浏览器能识别的样子。我们之前在文件夹中安装的webpack每次都要输入node_modules/.bin/webpack过于繁琐，可以在package.json进行如下修改

```
’script’: {
	‘start’: ‘webpack’
}
```
直接执行npm run start，可以发现结果一样。

## Loader
Loader是webpack一个很强大的功能。
###Babel
Babel可以让你使用ES6/ES7写代码而不用顾忌浏览器的问题，Babel可以帮你转换代码，首先要安装几个Babel库

```
npm i --save-dev babel-loader babel-core babel-preset-env
```
其中：

* babel-loader用于让webpack知道如何运行babel
* babel-core可以看做编译器，这个库知道如何解析代码
* babel-preset-env这个库可以根据环境的不同转换代码

更改webpack.config.js中的代码

```
modules.exports = {
	// ...
	module: {
		rules: [
			{
				// js文件才使用babel
				test: /\.js$/,
				// 使用哪个loader
				use: ‘babel-loader’,
				// 不包括路径
				exclude: /node_modules/,
				include: include: __dirname + 'src'
			}
		]
	}
}
配置**babel**有很多方式，推荐使用.babel文件管理
// ...babelrc
{
	“presets”: [“babel-preset-env”]
}
接着可以把自己的代码写成ES6（ ()=> {} ）
运行npm run start ,可以发现bundle.js代码被转换，代码正常输出。
```
### 处理图片
```
npm i --save-dev url-loader file-loader
```
创建一个images文件夹，放入图片，并在app下创建一个js文件处理图片，如下目录

![当前目录](https://camo.githubusercontent.com/62edda18a660864c09ad7ef38f75694ec5d45c0d/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f31352f3665393435616231646336623138383162663439666531373264346236336366)

```
// addImage.js
let samllImg = document.createElement(‘img’)
samllImg.src = require(‘../images/samll.jpg’)

document.body.appendChild(samllImg)

let bigImg = document.createElement(‘img’)
bigImg.src = require(‘../images/big.jpg’)

document.body.appendChild(bigImg)
```
接下来修改webpack.config.js代码

```
module.exports = {
	// ...
	module: {
		rules: [
			// ...
			{
				// 图片格式正则
				test: /\.(png|jpe?g|gif|svg)(\?.*)/,
				use: [
					{
						loader: ‘url-loader’,
						// 配置url-loader可选项
						options: {
							limit: 1000,
							// build/images/[图片名][hash].[图片格式]
							name: ‘iamges/[name].[hash].[ext]’
						}
					}
				]
			}
		]
	}
}
```
运行打包
![图片压缩](https://camo.githubusercontent.com/237a8363fb25b5f88474f5a8c3bbaeb23af3dbe2/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f31352f6234363062613735633932303532666664326466303337623736616637646462)
可以发现大图被提取，小图被打进bundle.js中
在浏览器发现，大图不见了，路径出了问题，要修改webpack.config.js代码

```
	module.exports = {
		entry: ‘./app/index.js’,
		output: {
			path: path.resolve(__dirname,’build’),
			filename: ‘bundle.js’,
			publishPath: ‘build/’
		}
	}
```
完美解决 ！
### 处理CSS文件
添加styles文件夹，新增addImage.css文件，新增样式代码
关于样式，我们先使用css-loader,style-loader库，前者可让CSS支持import，解析CSS，后者将解析的CSS通过标签形式插入HTML中，后者依赖前者。

```
npm i --save-dev css-loader style-loader
```
addImage文件引入css文件

```
import ‘../styles/addImage.css’
```
修改webpack.config.js代码

```
module.exports = {
	// ...
	module: {
		rules: [
			{
				test: /\.css$/,
				use: [‘style-loader’, {
					loader: ‘css-loader’,
					options: {
						modules: true
					}
				}]
			}
		]
	}
}
```
但是将CSS代码整合进JS文件也是有弊端的，大量的CSS代码会造成JS文件的大小变大，操作DOM也会造成性能上的问题，故我们需要使用extract-text-webpack-plugin插件将CSS文件打包为一个单独的文件
首先安装 npm i --save-dev extract-text-webpack-plugin

然后修改webpack.config.js代码

```
const ExtractTextPlugin = require(‘extract-text-webpack-plugin’)
module.exports = {
	// ...
	// 插件列表
	plugins: [
		// 输出文件路径
		new ExtractTextPlugin(‘css/[name].[hash].css’)
	]
}
```
运行npm run start可以发现CSS文件被单独打包了
![CSS单独打包](https://camo.githubusercontent.com/c842b773274807008820925ede5a02e3b6ca8920/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f31352f3334336134663465373239646338376630626136376136353231323864656133)

### 如何在项目中使用webpack
经过上述的优化，我们还是发现bundle.js很大，所以我们的目的就是将单个文件拆分成多个文件，优化项目。
####分离代码
首先我们可以考虑缓存机制。对于代码中依赖的库很少去主动升级版本，但是我们的代码一直在变，所以我们可以考虑将依赖的库和我们的代码分割开来，这样在用户下次使用应用的时候就可以避免重复下载没有变更的代码，那么既然依赖代码要提取，我们就需要变更下入口和出口的部分代码.

```
// pakage.json中的dependencies下的
const VENOR = [‘faker’,’lodash’,’react’,’react-dom’,’react-input-range’,’react-redux’,’redux’,’react-form’,’redux-thunk’]
module.exports = {
	// 之前我们使用单文件入口
	// entry同时也支持多文件入口，现在我们有两个
	// 一个是我们自己的代码，一个是依赖库代码
	entry: {
		// bundle和vendor都是自己随便起的，会映射到[name]中
		bundle: ‘./src/index.js’,
		vendor: VENOR
	},
	output: {
		path: path.join(__dirname, ‘dist’),
		filename: ‘[name].js’	
	},
	// ...
}
```
![打包](https://camo.githubusercontent.com/f6ff2f8acbd2937a4f14c6d47f463a390689dae2/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f392f31362f3337303437316562363366656561613732653836343135653339363134316533)
我们发现bundle引入了依赖库代码，需要抽取bundle中引入的代码。
#### 抽取共同代码
```
module.exports = {
	// ...
	output: {
		path: path.join(__dirname,’dist’),
		// 每次修改代码后修改文件名，缓存生效
		// [chunkhash]会自动根据文件是否更改更换哈希
		filename: ‘[name].[chunkhash].js’
	},
	plugins: [
		new webpack.optimiza.CommonsChunkPlugin({
		// vendor的意义和之前相同
		// mainfest文件是将每次打包都会更改的东西单独提取出来，保证没有更改的代码无需重新打包，加快打包速度
			names: [‘vendor’,’mainfest’],
			// 配合mainfest文件使用
			minChunks: Infinity
		})
	]	
}
```
