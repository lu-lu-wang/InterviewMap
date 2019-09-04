###git commit前检测husky与pre-commit

> 防止不规范的代码commit并push到远端，我们可以在git命令执行前用一些钩子来检测并阻止。现在主要有两种git钩子插件：husky(jq和nextjs在用)，pre-commit(antd在用)


####git钩子
>  cd .git/hooks
> ls -l
    
### husky
:: husky其实就是一个为git客户端增加hook的工具。将其安装到所在仓库的过程中它会自动在.git/目录下增加相应的钩子，实现在pre-commit阶段就执行一系列流程保证每一个commit的正确性。部分在cd commit stage执行的命令挪到本地执行，比如lint检查、比如单元测试。
husky能防止不规范代码被commit、push、merge等等
首先安装husky
> npm i husky --save-dev
  

编辑package.json文件：添加pre-commit钩子 

`
{ 
	“script”: {
		“precommit”: “webpack --config ./web/webpack.config.js”
	}
}
`
##集成
通过husky为prettier在pre-commit加个钩子，不论你什么格式，只要想提交，就必须格式化成prettier要求的样子

1. 添加prettier依赖

`
	yarn add prettier --dev --exact
` 


2. 在commit时执行prettier 

`
	yarn add pretty-quick husky --dev
`

修改package.json添加pre-commit钩子

`
	{
		“scripts”: {
			“precommit”: “pretty-quick --staged”
		}
	}
`
