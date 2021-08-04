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



## 04 el与data的两种写法

- data与el的2种写法
  - 1.el有2种写法
    - (1).new Vue时候配置el属性。
    - (2).先创建Vue实例，随后再通过vm.$mount('#root')指定el的值。（挂载）
  - 2.data有2种写法
    - (1).对象式
    - (2).函数式
    - 如何选择：目前哪种写法都可以，以后学习到组件时，data必须使用函数式，否则会报错。
  - 3.一个重要的原则：由Vue管理的函数，一定不要写箭头函数，一旦写了箭头函数，this就不再是Vue实例了。

```html
<!DOCTYPE html>
<html>
   <head>
      <meta charset="UTF-8" />
      <title>el与data的两种写法</title>
      <!-- 引入Vue -->
      <script type="text/javascript" src="../js/vue.js"></script>
   </head>
   <body>
      <!-- 
         data与el的2种写法
               1.el有2种写法
                           (1).new Vue时候配置el属性。
                           (2).先创建Vue实例，随后再通过vm.$mount('#root')指定el的值。
               2.data有2种写法
                           (1).对象式
                           (2).函数式
                           如何选择：目前哪种写法都可以，以后学习到组件时，data必须使用函数式，否则会报错。
               3.一个重要的原则：
                           由Vue管理的函数，一定不要写箭头函数，一旦写了箭头函数，this就不再是Vue实例了。
      -->
      <!-- 准备好一个容器-->
      <div id="root">
         <h1>你好，{{name}}</h1>
      </div>
   </body>

   <script type="text/javascript">
      Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

      //el的两种写法
      /* const v = new Vue({
         //el:'#root', //第一种写法
         data:{
            name:'尚硅谷'
         }
      })
      console.log(v)
      v.$mount('#root') //第二种写法 */

      //data的两种写法
      new Vue({
         el:'#root',
         //data的第一种写法：对象式
         /* data:{
            name:'尚硅谷'
         } */

         //data的第二种写法：函数式
         data(){
            console.log('@@@',this) //此处的this是Vue实例对象
            return{
               name:'尚硅谷'
            }
         }
      })
   </script>
</html>
```



## 05 MVVM模型

- MVVM模型

  - M：模型(Model) ：data中的数据
  - V：视图(View) ：模板代码
  - VM：视图模型(ViewModel)：Vue实例

- 观察发现：

  - data中所有的属性，最后都出现在了vm身上。
  - vm身上所有的属性 及 Vue原型上所有属性，在Vue模板中都可以直接使用。

  ![]()



## 数据代理

```html
<!DOCTYPE html>
<html>
   <head>
      <meta charset="UTF-8" />
      <title>回顾Object.defineproperty方法</title>
   </head>
   <body>
      <script type="text/javascript" >
         let number = 18
         let person = {
            name:'张三',
            sex:'男',
         }

         Object.defineProperty(person,'age',{
            // value:18,
            // enumerable:true, //控制属性是否可以枚举，默认值是false
            // writable:true, //控制属性是否可以被修改，默认值是false
            // configurable:true //控制属性是否可以被删除，默认值是false

            //当有人读取person的age属性时，get函数(getter)就会被调用，且返回值就是age的值
            get(){
               console.log('有人读取age属性了')
               return number
            },

            //当有人修改person的age属性时，set函数(setter)就会被调用，且会收到修改的具体值
            set(value){
               console.log('有人修改了age属性，且值是',value)
               number = value
            }

         })

         // console.log(Object.keys(person))

         console.log(person)
      </script>
   </body>
</html>
```

```html
<!DOCTYPE html>
<html>
   <head>
      <meta charset="UTF-8" />
      <title>何为数据代理</title>
   </head>
   <body>
      <!-- 数据代理：通过一个对象代理对另一个对象中属性的操作（读/写）-->
      <script type="text/javascript" >
         let obj = {x:100}
         let obj2 = {y:200}

         Object.defineProperty(obj2,'x',{
            get(){
               return obj.x
            },
            set(value){
               obj.x = value
            }
         })
      </script>
   </body>
</html>
```

- Vue中的数据代理：通过vm对象来代理data对象中属性的操作（读/写）
- Vue中数据代理的好处：更加方便的操作data中的数据
- 基本原理：通过Object.defineProperty()把data对象中所有属性添加到vm上。为每一个添加到vm上的属性，都指定一个getter/setter。在getter/setter内部去操作（读/写）data中对应的属性。

```html
<!DOCTYPE html>
<html>
   <head>
      <meta charset="UTF-8" />
      <title>Vue中的数据代理</title>
      <!-- 引入Vue -->
      <script type="text/javascript" src="../js/vue.js"></script>
   </head>
   <body>
      <!-- 
            1.Vue中的数据代理：
                     通过vm对象来代理data对象中属性的操作（读/写）
            2.Vue中数据代理的好处：
                     更加方便的操作data中的数据
            3.基本原理：
                     通过Object.defineProperty()把data对象中所有属性添加到vm上。
                     为每一个添加到vm上的属性，都指定一个getter/setter。
                     在getter/setter内部去操作（读/写）data中对应的属性。
       -->
      <!-- 准备好一个容器-->
      <div id="root">
         <h2>学校名称：{{name}}</h2>
         <h2>学校地址：{{address}}</h2>
      </div>
   </body>

   <script type="text/javascript">
      Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。
      
      const vm = new Vue({
         el:'#root',
         data:{
            name:'尚硅谷',
            address:'宏福科技园'
         }
      })
   </script>
</html>
```















