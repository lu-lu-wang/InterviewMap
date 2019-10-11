###写一个npm包并上传至npm仓库
1. 首先要有一个npm账号（去npm官网注册一个即可）
2. 新建一个文件夹，并初始化

`  mkdir demo // 新建文件
	cd demo   // 进入文件
	npm init  // 初始化基本参数
`

3. 初始化需输入信息：
 1. name: 包名称（不可输入大写字母、空格、下划线）
 2. version: 包版本（默认1.0.0）
 3. description: 项目介绍
 4. mian: 入口文件（默认Index.js）
 5. scripts: 包含各种脚本执行命令
 6. test: 测试命令。。。

4. 之后生成package.json配置

`  
	   
	   {
		  “name”: “share-package”,
		  “version”: “1.0.13”,
		  “description”: “a share demo”,
		  “main”: “index.js”,
		  “scripts”: {
		    “test”: “npm run test”,
		    “build”: “tsc”
		  },
		  “keywords”: [
		    “share”
		  ],
		  “author”: “wanglu@sunmi.com”,
		  “license”: “ISC”
	  }
`

5. 可在文件夹里创建index.js

`

	!function(){
	  console.log(`这是引入的包入口`)
	}()
`

6. 登录自己的npm账号、密码(登录地址要是官方源地址（非淘宝镜像）)

` npm login`

7. 发包

`npm publish`

8. 成功后即可去npm官网搜索即可

`
	npm unpublish --force // 删除已经上传的包
`

