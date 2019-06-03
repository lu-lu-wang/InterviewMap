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

