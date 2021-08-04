# Vue

## 01 初识Vue：
   1.想让Vue工作，就必须创建一个Vue实例，且要传入一个配置对象；
   2.root容器里的代码依然符合html规范，只不过混入了一些特殊的Vue语法；
   3.root容器里的代码被称为【Vue模板】；
   4.Vue实例和容器是一一对应的；
   5.真实开发中只有一个Vue实例，并且会配合着组件一起使用；
   6.{{xxx}}中的xxx要写js表达式，且xxx可以自动读取到data中的所有属性；
   7.一旦data中的数据发生改变，那么页面中用到该数据的地方也会自动更新；

- 注意区分：js表达式 和 js代码(语句)
  - 1.表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方：
    - (1). a
    - (2). a+b
    - (3). demo(1)
    - (4). x === y ? 'a' : 'b'
  - 2.js代码(语句)
    - (1). if(){}
    - (2). for(){}

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>初识Vue</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 
			初识Vue：
				1.想让Vue工作，就必须创建一个Vue实例，且要传入一个配置对象；
				2.root容器里的代码依然符合html规范，只不过混入了一些特殊的Vue语法；
				3.root容器里的代码被称为【Vue模板】；
				4.Vue实例和容器是一一对应的；
				5.真实开发中只有一个Vue实例，并且会配合着组件一起使用；
				6.{{xxx}}中的xxx要写js表达式，且xxx可以自动读取到data中的所有属性；
				7.一旦data中的数据发生改变，那么页面中用到该数据的地方也会自动更新；

				注意区分：js表达式 和 js代码(语句)
						1.表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方：
									(1). a
									(2). a+b
									(3). demo(1)
									(4). x === y ? 'a' : 'b'

						2.js代码(语句)
									(1). if(){}
									(2). for(){}
		-->

		<!-- 准备好一个容器 -->
		<div id="demo">
			<h1>Hello，{{name.toUpperCase()}}，{{address}}</h1>
		</div>

		<script type="text/javascript" >
			Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

			//创建Vue实例
			new Vue({
				el:'#demo', //el用于指定当前Vue实例为哪个容器服务，值通常为css选择器字符串。
				data:{ //data中用于存储数据，数据供el所指定的容器去使用，值我们暂时先写成一个对象。
					name:'atguigu',
					address:'北京'
				}
			})

		</script>
	</body>
</html>
```



## 02 Vue的模板语法

- Vue模板语法有2大类：
  - 1.插值语法：
    - 功能：用于解析标签体内容。
    - 写法：{{xxx}}，xxx是js表达式，且可以直接读取到data中的所有属性。
    
  - 2.指令语法：

    - 功能：用于解析标签（包括：标签属性、标签体内容、绑定事件.....）。举例：v-bind:href="xxx" 或  简写为 :href="xxx"，xxx同样要写js表达式，且可以直接读取到data中的所有属性。
    - 备注：Vue中有很多的指令，且形式都是：v-????，此处我们只是拿v-bind举个例子。

    ```html
    <!DOCTYPE html>
    <html>
    	<head>
    		<meta charset="UTF-8" />
    		<title>模板语法</title>
    		<!-- 引入Vue -->
    		<script type="text/javascript" src="../js/vue.js"></script>
    	</head>
    	<body>
    		<!-- 
    				Vue模板语法有2大类：
    					1.插值语法：
    							功能：用于解析标签体内容。
    							写法：{{xxx}}，xxx是js表达式，且可以直接读取到data中的所有属性。
    					2.指令语法：
    							功能：用于解析标签（包括：标签属性、标签体内容、绑定事件.....）。
    							举例：v-bind:href="xxx" 或  简写为 :href="xxx"，xxx同样要写js表达式，
    									 且可以直接读取到data中的所有属性。
    							备注：Vue中有很多的指令，且形式都是：v-????，此处我们只是拿v-bind举个例子。
    
    		 -->
    		<!-- 准备好一个容器-->
    		<div id="root">
    			<h1>插值语法</h1>
    			<h3>你好，{{name}}</h3>
    			<hr/>
    			<h1>指令语法</h1>
    			<a v-bind:href="school.url.toUpperCase()" x="hello">点我去{{school.name}}学习1</a>
    			<a :href="school.url" x="hello">点我去{{school.name}}学习2</a>
    		</div>
    	</body>
    
    	<script type="text/javascript">
    		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。
    
    		new Vue({
    			el:'#root',
    			data:{
    				name:'jack',
    				school:{
    					name:'尚硅谷',
    					url:'http://www.atguigu.com',
    				}
    			}
    		})
    	</script>
    </html>
    ```



## 03 数据绑定

Vue中有2种数据绑定的方式：

- 1.单向绑定(v-bind)：数据只能从data流向页面。
- 2.双向绑定(v-model)：数据不仅能从data流向页面，还可以从页面流向data。

备注：

1.双向绑定一般都应用在表单类元素上（如：input、select等）

2.v-model:value 可以简写为 v-model，因为v-model默认收集的就是value值。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>数据绑定</title>
		<!-- 引入Vue -->
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<!-- 
			Vue中有2种数据绑定的方式：
					1.单向绑定(v-bind)：数据只能从data流向页面。
					2.双向绑定(v-model)：数据不仅能从data流向页面，还可以从页面流向data。
						备注：
								1.双向绑定一般都应用在表单类元素上（如：input、select等）
								2.v-model:value 可以简写为 v-model，因为v-model默认收集的就是value值。
		 -->
		<!-- 准备好一个容器-->
		<div id="root">
			<!-- 普通写法 -->
			<!-- 单向数据绑定：<input type="text" v-bind:value="name"><br/>
			双向数据绑定：<input type="text" v-model:value="name"><br/> -->

			<!-- 简写 -->
			单向数据绑定：<input type="text" :value="name"><br/>
			双向数据绑定：<input type="text" v-model="name"><br/>

			<!-- 如下代码是错误的，因为v-model只能应用在表单类元素（输入类元素）上 -->
			<!-- <h2 v-model:x="name">你好啊</h2> -->
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

		new Vue({
			el:'#root',
			data:{
				name:'尚硅谷'
			}
		})
	</script>
</html>
```































