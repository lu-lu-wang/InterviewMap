### babel之配置文件.babelrc
babel: js语法编译器
[]()

项目工程脚手架中，一般会使用.babelrc文件，通过配置一些参数配合webpack进行打包压缩。

#### babel转译器
在.babelrc配置文件中，主要对预设presets和插件plugins进行配置。

1. 语法转译器。主要对js最新语法糖进行编译，并不负责转义js新增的api和全局对象。例如let/const就可以被编译，而includes/Object.assign等不能被编译，常用到的转译器包有：babel-preset-env,babel-preset-es2015等。在实际开发中可以值选用babel-preset-env来替代余下的：

`
	{
		“presets”: [“env”, {
			“modules”: false
		}],
		“stage-2”
	}
`

2. 补丁转译器。主要负责转义js新增的api和全局对象，例如：bable-plugin-transform-runtime这个插件能够编译Object.assign,同时可引入bable-polufill进一步对includes这类用法保证在浏览器的兼容性。Objext.assign会被编译成：

`__WEBPACK_IMPORTED_MODULE_1_babel_runtime_core_js_object_assign___default()
`

3. jsx和flow插件，这类转译器用来转义jsx语法和移除类型声明的，使用React的时候你将用到，转译器名称：babel-preset-react
####创建预设（presets）
作用是为babel安装指定插件，主要通过npm安装babel-preset-xx插件来配合使用，安装插件后，在.babelrc文件里配置
`
	{
		“presets”: [
			[“env”, options],
			“stage-2”
		]
	}
`

#### stage-2配置
babel主要提供以下几种转译包：
`
 babel-preset-stage-0(配置项：stage-0)
 babel-preset-stage-1(配置项：stage-1)
 babel-preset-stage-2(配置项：stage-2)
 babel-preset-stage-3(配置项：stage-3)
`
不同阶段的转译器之间是包含关系，preset-stage-0转译器包含了preset-stage-1所有功能还增加了transform-do-expressions插件和reansform-function-bind插件，同样preset-stage-1转译器除了包含preset-stage-2的全部功能还增加了一些额外功能。
#### options配置介绍
官方推荐使用**babel-preset-env**来替代一些插件包安装：
`{
	“target”: {
		“chrome”: 52,
		“browsers”: [“last 2 version”, “safari 7”],
		“node”: “6.10”
	}
	“modules”: false
}
`
targets可以指定兼容浏览器版本，如果设置了browsers，那么就会覆盖targets原本对浏览器的限制配置。
tagets.node正对node版本进行编译
modules通常会设置为false,因为默认都是支持CommonJS规范，同时还有其他配参：”amd” | “umd” | “systemjs” | “commonjs”

#### 插件（plugins）
插件配置项同预设配置项一样，需要搭配babel相应的插件进行配置，可以选择配置插件来满足单个需求：
`
	{
		“plugins”: [
			“check-es2015-constants”,
			“es2015-arrow-functions”,
			“es2015-block-scoped-function”
			...
		]
	}
`
但是这些插件从维护到书写极为麻烦，后来官方同意推荐使用env，全部替代了单一插件功能，可以简化配置（babel-preset-env）:
`
	{
		“presets”: [
			“es2015”
		]
	}
`
常用插件： 
`
	{
		“plugins”: [
			“syntax-dynamic-import”, [“transform-runtime”]
		]
	}
`
#### transform-runtime
解决全局对象或者全局对象编译不足的情况，才出现transform-runtime这个插件，但是它只会对es6语法转换，不会对新api进行转换，使用transform-runtime后，可以减少内部全局函数定义，从结构上看遵循webpack模块化思想



###组件库按需加载 babel-plugin-import
antd-design为例：
antd主要借助了自己写的babel插件*babel-plugin-import*。
其原理：在babel转码的时候，把整个‘antd’引用，变为‘antd/lib/button’具体模块的引用，这样webpack手机依赖module就不是整个antd,而是里面的button

```{‘libraryName’: ‘antd’}```
![原理代码](https://images2018.cnblogs.com/blog/106679/201802/106679-20180219162629682-1533370487.png)
我们发现.babelrc是json文件不支持function（babel7支持），我们可以这样写

```
{
	‘presets’: [‘./.babelrc.js’]
}
```
原来.babelrc的配置放到.babelrc.js，自己处理下map对应关系

```
module.exports = {
	‘presets’: [‘react’,’es2015’,’stage-0’],
	‘plugins’: [
		‘transform-runtime’,
		‘lodash’,
		‘transform-decorators-legacy’,
		‘jsx-control-statements’,
		[‘transform-react-remove-prop-types’, {
			‘removeImport’: true,
			‘mode’: ‘remove’
		}],
		[‘import’,{
			‘libraryName’: ‘my-react’,
			camel2UnderlineComponentName: false,
			camel2DashComponentName: false,
			customName: function (name) {
				if (!map[name]) {
					console.log(name)
				}
				return ‘my-react/src${map[name]}’
			}
		}]
	]
}
```

