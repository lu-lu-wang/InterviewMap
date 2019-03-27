# antd之Form表单
form: 具有数据收集、校验和提交功能的表单，包含单选框、复选框、输入框、下拉框等元素。

表单表现方式有三种：

1.水平排列：标签和表单控件水平排列（默认）

2.垂直排列：标签和表单控件垂直排列

3.行内排列：表单项水平行内排列

表单域：

表单一定会包含表单域，表单域可以是输入控件，标准表单域，标签或者下拉菜单，文本域等，这里封装表单域：Form.item

<Form.Item {...props}>{children}</Form.Item>

演示：

1.水平登录栏，常用于顶部导航栏中


import { From, Icon, Input, Button } from ‘antd’

const FormItem = From.Item

function hasErrors (fieldsError) {

	return Object.keys(fieldsError).some(field => fieldsErrors[filed])
}


class HorizontalLoginForm extents react.Component {


componentDidMount () {

	this.props.form.validdataFileds()

}


}

handleSubmit = (e) => {

 	e.preventDefault()

	this.props.form.validataFileds((err, values) => {	

	if(!err){
		console.log(‘Reaceived values of from:’, values)
}
	})	
}

render () {
	

	const { getFiledDecorator, getFiledsError, getFiledError, isFieldTouched } = this.props.form

	const userNameError = isFieldTouched(‘userName’) && getFieldError(‘userName’)

	const passwordError = isFieldTouched(‘password’) && getFieldError(‘password’)

	return (
	<Form layout="inline" onSubmit={this.handleSubmit}>
		<FormItem validateStatus={‘userNameError?’error’:’’’}>
		{getFieldDecorator(‘userName’,{ rules: 
		[{requeired:true, message:’Please input your userName!’}],})(<Input prefix={<Icon type=“user”>}>)}
</FormItem>
</Form>
)
}

const WrappedHorizontalLoginForm = Form.create()(HorizontalLoginForm)

### 表单数据存储于上层组件
通过使用onFiledsChange与mapPropsToFields,可以吧表单的数据存储到上层组件或者redux、dva。
注意：mapPropsToFields里面返回的表单数据必须使用Form.creactFormField包装
使用方法：class Demo extends react.Component{}
Demo = Form.creact({})(Demo)

option配置如下：

mapPropsToFields:把父组件的属性映射到表单项上，需要对返回值中的表单域数据用Form.createFormField标记
validateMessages: 默认校验信息，用于把错误信息改为中文等，格式与newMessages返回一致
onFieldsChange:当Form.Item子节点的值发生改变时触发，可以把对应的值转存到Redux store中去
onValuesChange: 任一表单域的值发生改变时的回调

经过Form.Item之后如果要拿到ref，可以使用rc-from提供的wrappedComponentRef

经过Form.creact包装的组件将自带this.props.form属性，this.props.form提供API如下：
注意：使用getFieldsValue getFieldValue,setFieldValue等时，应确保对应的field已经使用getFieldDecorator注册过了

getFiledDecorator: 用于和表单双向绑定

getFieldError: 获取某个输入控件的error

getFieldsError: 获取一组输入控件的error，如不传入参数，则获取全部组件的error

getFieldsValue: 获取一组控件的值，如不传入参数，则获取全部组件的值：function([fieldName:string[]])


isFieldsTouched: 判断是否任一输入控件经历过getFieldDecorator的值收集时机options.trigger

isFieldValidating: 判断一个输入控件是否在校验状态

resetFields: 重置一组输入控件的值与状态，如不传入参数，则重置所有组件

validateFields: 校验并获取一组输入域的值域与error，若fieldNames参数为空，则校验全部组件 

form.validateFields((err) => {if (err) {
          console.error(err)
          return null
 		}
 	this.props.onSubmit(form.getFieldsValue())
 }
 
Form.createFormField: 用于标记mapPropsToFields返回的表单域数据。
this.props.form.getFieldDecorator(id,option)
经过getFieldDecorator包装的控件，表单控件会自动添加value，onChange,数据同步将被Form接管，这会导致：

1.你不需要也不应该用onChange来同步，但还是可以继续监听onChange等事件。

2.你不能用控件value，defaultValue等属性来设置表单域的值，默认值可以用getFieldDecorator里的initalValue.

3.你不应该用setState，可以使用this.props.form.setFieldsValue来动态改变表单值。

特备注意：

1.getFieldDecorator不能用于装饰纯函数组件

2.如果使用的是react@<15.3.0,则getFieldDecorator调用不能位于纯函数组件中。

getFieldDecorator(id,option)参数：

{getFieldDecorator('member[0].name.firstname', {})(<input/>)}

id: 必填输入控件唯一标志，支持嵌套式的写法

options.getValueFormEvent: 可以把onChnage的参数转化为控件的值

options.initialValue 子节点的初始值，类型、可选值均由子节点决定，

options.rules: 校验规则

options.trigger: 收集子节点的值得时机

options.validateFirst: 当某一规则校验不通过时，是否停止剩下的规则校验

options.validateTrigger: 校验子节点值的时机

options.valuePropName: 子节点的值的属性，如switch的是checked

Form.Item

一个Form.Item建议只放一个getFieldDecorator修饰过的child，当有多个被修饰过的child时，help，required validateState无法自动生成
参数：
colon: 配合label属性使用，表示是否显示lable后面的冒号

extra: 额外的提示信息，和help类似，当需要错误信息和提示文案同时出现时，可以使用

hasFeedback: 配合validateStatus属性使用，展示校验状态图标，建议只配合input组件使用

label: lable标签的文本

labelCol: label标签布局，同<Col>组件，设置span offest值，如{span:3,offset:12}或sm:{span:3,offset:12}

required: 是否必填，如不设置，则会根据校验规则自动生成

validateStatus: 校验状态，如不设置，则会根据校验规则自动生成，可选：‘success’,’waring’,’error’,’validating’

wrapperCol: 需要为输入控件设置布局样式时，使用该属性，用法同labelCol(一个是lable的栅格，一个是控件的栅格)
