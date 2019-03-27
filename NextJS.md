### NextJS
安装： npm install --save next react react-domz
在你的package.json文件添加：

		{
			‘scripts’: {
				‘dev’: ‘next’,
				‘build’: ‘next build’,
				‘start’: ‘next start’
			}
		}

接下来，大部分时间都交由文件系统来处理，每个.js文件都变成一个自动处理和渲染的路由。在项目中新建./pages/index.js
	
	export default () => <div>Welcome to next.js</div>

然后在控制台输入npm run dev 命令，打开locallhost:3000即可看到程序已经运行（或者使用 npm run dev -- -p<your port here>指定端口运行程序）

注意：1.next框架自带<head>标签，作为当前页面的<head>,如果在组件内定义了<Head>则定义<Head>内的元素将会被追加到框架自带的<head>标签中
2.每个组件自定义的<Head>内容只会应用在各自的页面上，子组件内定义的<Head>也会追加到当前页面的<head>内，如果有重复定义的标签或属性，则子组件覆盖父组件，位于文档更后面的组件覆盖更前面的组件。

### 数据获取及组件的生命周期
你可以通过导出一个基于React.Component的组件来获取状态（state）、生命周期或者初始数据（而不是一个无状态函数stateless）

	import React from ‘react’
	export default class extends React.Component {
		static async getInitialProps({ req }){
			const userAgent = req.headers[‘user-agent’]: navigator.userAgent
			return { userAgent }
		}
		return () {
			<div>Hello World {this.props.userAgent}</div>
		}
	}
	当加载页面获取数据的时候，我们使用了一个异步aysnc的静态方法getInitialProps，此静态方法能够获取所有数据，并将其解析成一个js对象，然后将其最为属性附加到props对象上。当初始化页面时候，getInitialProps只会在服务器端执行，而当通过Link组件或者使用命令路由API来将页面导航到另一个路由的时候，此方法就只会在客户端执行。
	！！！注意： getInitialProps不能在子组件上使用，只能应用于当前页面的顶层组件。
	
	你也可以为无状态（stateless）组件自定义getInitialProps生命周期方法：
	const Page = ({ stars }) => 
	<div>
		Next stars: {stars}
	</div>
	Page.getInitialProps = async ({ req }) => {
		const res = await fetch(‘https://api.github.com/repos/zeit/next.js’)
		const json =  await res.json()
		return { stars: json.stargazers_count}
	}
	export default Page

#### getInitialProps接收的上下文对象包含以下属性：
1.  pathname - URL的 path部分
2.  query - URL的 query string 部分，并且其已经被解析成了一个对象
3. asPath - 在浏览器上展示的实际路径（包括query字符串）
4. req - HTTP request 对象（只存在于服务器端）
5. res - HTTP respones 对象（只存在于服务器端）
6. jsonPageRes - 获取相应数据对象Fetch Response(只存在于客户端)
7. err - 渲染时发生错误抛出的错误对象

### Link
可以通过<Link>组件来实现客户端在两个路由间的切换功能

	import Link from ’next/link’
	
	export default () => 
	<div>
		Click{‘ ’}
		<Link href=‘/about’>
			<a>href</a>
		</Link>{‘ ‘}
		to read more
	</div>
	// pages/about.js
	export default () => <p> Welcome to About!</p>
	注意： 可以使用<Link prefetch>来让页面在后台同时获取和预加载，已获得最佳的页面加载性能
	客户端路由行为与浏览器完全相同：
	1. 获取组件
	2. 如果组件定义了getInitialProps，那么进行数据的获取，如果抛出异常，则渲染_error.js
	3. 在步骤1和步骤2完成后，pushState开始执行，接着新组件将会被渲染。

#### 每一个顶层组件都会接收到一个url属性，其中包括以下API：
	· pathname - 不包括query字符串在内的当前链接地址的path字符串（即pathname）
	· query - 当前链接地址的query字符串，已经被解析为对象，默认为{}
	· asPath - 在浏览器地址栏显示的当前页面的实际地址（包括query字符串）
	· push(url, as=url) - 通过pushState来跳转路由给定的url
	· replace(url, as=url) - 通过replaceState来将当前路由替换到给定的路由地址url上。
	push以及replace的第二个参数as提供了额外的配置项，当你在服务器上配置了自定义路由的话，那么此参数就会发挥作用。

#### 替换(replace)而非追加(push)路由url
<link>组件默认将新的URL追加（push）到路由栈中，但你可以使用replace属性来避免此追加动作（直接替换掉当前路由）。

	import Link from ‘next/link’
	
	export default () =>
	<div>
		Click{‘ ’}
		<Link href=“/about” repalce>
			<a>href</a>
			to read me
		</Link>Click{‘’}
	</div>

#### 命令式路由
你可以使用next/router来实现客户端侧的页面切换

	import Router from ‘next/router’
	
	export default () => 
	<div>
	Click<span onClick={() => Router.push(‘/about’)}>here</span>to read more
	</div>

#### 使用高阶函数HOC
如果你想在应用的任何组件都能获取到router对象，那么你可以使用withRouter高阶函数

	import { withRouter } from ‘next/router’
	
	const ActiveLink = ({children, router, href}) => 
	const style = {
		marginRight: 20,
		color: router,pathname === href ? ‘red’, ‘black’
		}
		const handleClick = (e) => {
			e.preventDefault()
			router.push(href)
		}
		return (
			<a href={href} onClick={handleClick} style={style}>
				{children}
			</a>
		)
	}
	export default withRouter(ActiveLink)

#### 预获取页面Prefetching Page
Next.js自带允许你预获取（prefetch）页面的API，因为Next.js在服务器端渲染页面，所以应用的所有将来可能发生交互的相关链路路径可以在瞬间完成交互，事实上Next.js可以通过预下载功能来达到一个绝佳的加载性能。由于Next.js只会预加载js代码，所以在页面加载的时候，你需要花时间来等待数据的获取。

	



	
	


		
