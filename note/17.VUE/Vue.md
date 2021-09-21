# 基础

## MVVM模型

1. M: 模型 data中数据
2. V: View 模板代码
3. VM: 视图模型(ViewModewl) Vue实例

简而言之,Vue串联Dom对象与Model数据之间. 将数据挂载到Dom,将Dom时间通知Model

------

data中的所有属性都会出现在Vue实例中

Vue实例中所有的属性以及原型上的所有属性,在Vue模板中都可以直接使用

## Vue实例

创建Vue实例

```javascript
//创建Vue实例
new Vue()
//需要传入一个对象
new Vue({
    el:'#demo', //el用于指定当前Vue实例为哪个容器服务，值通常为css选择器字符串。
    data:{ //data中用于存储数据，数据供el所指定的容器去使用，值我们暂时先写成一个对象。
        name:'atguigu',
        address:'北京'
    }
})
```

1. 想让Vue工作，就必须创建一个Vue实例，且要传入一个配置对象；
2. root容器里的代码依然符合html规范，只不过混入了一些特殊的Vue语法；
3. root容器里的代码被称为【Vue模板】；
4. Vue实例和容器是一一对应的；
5. 真实开发中只有一个Vue实例，并且会配合着组件一起使用；
6. {{xxx}}中的xxx要写js表达式，且xxx可以自动读取到data中的所有属性；
7. 一旦data中的数据发生改变，那么页面中用到该数据的地方也会自动更新；
   - 注意区分：js表达式 和 js代码(语句)
     - 表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方：
       -  a
       - a+b
       - demo(1)
       - x === y ? 'a' : 'b'
   - js代码(语句)
     -  if(){}
     - for(){}

### el

> 当一个Vue对象的el值的选择器对应的容器出现的多个容器时,Vue会选择Dom元素的第一个.仅会选择一个
>
> 当有多个Vue对象,并el选择同一个容器时买第二个Vue对象无效果, 一个容器只会被一个Vue接管

```javascript
//容器
<div id="demo"> <div/> //容器内的代码依然符合html规范
new Vue({
    el:'#demo', //el用于指定当前Vue实例为哪个容器服务，值通常为css选择器字符串。 #demo .demo等
})
```

Vue绑定容器有两种实现方式

- el: "选择器"
- Vue({...}).$mount("选择器");

二者使用任何一种方式都可以

### data

```javascript
----------------------方式1
new Vue({
    data:{ //data中用于存储数据，数据供el所指定的容器去使用，值我们暂时先写成一个对象。
        name:'atguigu',
        address:'北京'
    }
})
-----------------------函数式data
new Vue({
    data:function(){
        return { 
            name:'atguigu',
            address:'北京'
            }
    }
})
// 简写方式. 不建议再简写为箭头函数
new Vue({
    data(){
        return { 
            name:'atguigu',
            address:'北京'
            }
    }
})
```

data定义属性有两种方式

- 定义对象, 在组件下不建议使用,
- **使用函数返回对象,** 不可以使用箭头函数,因为箭头函数的this属于Window而不是Vue

优先使用函数方式定义data

## 模板语法

### 插值语法

- 用于解析标签体内容
- 写法{{xxx}},xxx是JS表达式或者Vue实例data中的属性

```java
<div id="demo">{{name}}<div/> //插值表达式 可以写变量名或者JS表达式
new Vue({
    el:'#demo', 
    data: {
        name: '插值' //变量值
    }   
})
```

### 指令语法

#### v-bind

单项数据绑定

- 用于`解析标签`,包括`标签属性,标签体内容,绑定事件`
- 举例: v-bind:href="xxx" xxx支持Js表达式与data中的属性
- 简写方式`v-bind`->`:    ` 范例 v-bind:href:="xxx" -> :href="xxx" 

```javascript
<div id="demo" :src="name"  v-bind:href="xxx"><div/> //指令语法
new Vue({
    el:'#demo', 
    data: {
        name: '插值' //变量值
    }   
})
```

 

#### v-model

双向数据绑定

> `v-model`为`v-model:value`的简写方式

- v-model双向数据绑定只能卸载表单类元素上
- v-model收集的是表单元素的value值, 其完整写法为`v-model:value` 简写为 `v-model`

## 数据绑定/代理

数据代理

### 理解数据绑定/代理

数据代理: 通过一个对象代理对另一个对象中属性的操作

### 回顾数据绑定/代理

```javascript
<script type="text/javascript" >
			let number = 18
			let person = {
				name:'张三',
				sex:'男',
			}

			Object.defineProperty(person,'age',{
				// value:18,  //value的值通过gtter获取时,value可以省略
				// enumerable:true, //控制属性是否可以枚举，默认值是false
				// writable:true, //控制属性是否可以被修改，默认值是false
				// configurable:true //控制属性是否可以被删除，默认值是false

				//当有人读取person的age属性时，get函数(getter)就会被调用，且返回值就是age的值
                //可以实现动态的读取number值赋予age, 不同于直接在person中定义,仅在首次读取number值
                //每次获取age值都会调用getter实时读取number(可能被修改,读取被修改后的)
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
```

## Vue中数据绑定/代理

Vue实例中多处用到数据代理

> 配置在data中的属性/方法会被数据劫持数据代理

Vue将Options对象中的属性存储在Vue中_开头的属性中.

1. Vue中的数据代理：
   - 通过vm对象来代理data对象中属性的操作（读/写）
2. Vue中数据代理的好处：
   - 更加方便的操作data中的数据
3. 基本原理：
   - 通过Object.defineProperty()把data对象中所有属性添加到vm上。
   - 每一个添加到vm上的属性，都指定一个getter/setter。
   - 在getter/setter内部去操作（读/写）data中对应的属性。
   - vm.name -> 调用了getter -> _data.name //_data 存储的是 Options中data

```javascript
data.name = xxx -> 调用setter -> _data.name = setter结果
```

![image-20210921002700520](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210921003423.png)

## 事件处理

`v-on:事件='方法名'` Vue通过v-on绑定事件; 简写为`@`

> 绑定的方法属于Vue管理的函数,不建议使用箭头函数
>
> 配置在method中的不会被数据劫持数据代理

### 基本使用

1. 使用v-on:xxx 或 @xxx 绑定事件，其中xxx是事件名；
2. 事件的回调需要配置在methods对象中，最终会在vm上；
3. methods中配置的函数，不要用箭头函数！否则this就不是vm了；
4. methods中配置的函数，都是被Vue所管理的函数，this的指向是vm 或 组件实例对象；
5. @click="demo" 和 @click="demo($event)" 效果一致，但后者可以传参；

```html
	<body>
		<div id="root">
			<!-- <button v-on:click="showInfo">点我提示信息</button> -->
			<button @click="showInfo1">点我提示信息1（不传参）</button> //默认会传递event对象
			<button @click="showInfo2($event,66)">点我提示信息2（传参）</button> //$event是vue的占位符,告诉vue需要传递事件对象
		</div>
	</body>
<script type="text/javascript">
    Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

    const vm = new Vue({
        el:'#root',
        data:{
            name:'尚硅谷',
        },
        methods:{
            showInfo1(event){  //event事件对象
                alert('同学你好！')
            },
            showInfo2(event,number){ //事件对象,参数
                alert('同学你好！！')
            }
        }
    })
</script>
```

### 事件修饰符

1. 阻止事件的默认行为`prevent`

```javascript
<!-- 阻止默认事件（常用） -->
<a href="http://www.atguigu.com" @click.prevent="showInfo">点我提示信息</a> 
//会调用showInfo但是 a标签的默认行为不会跳转
```

2. 阻止事件冒泡`stop`

```javascript
<!-- 阻止事件冒泡（常用） -->
<div class="demo1" @click="showInfo">
    <button @click.stop="showInfo">点我提示信息</button>
<!-- 修饰符可以连续写 -->
     <!-- <a href="http://www.atguigu.com" @click.prevent.stop="showInfo">点我提示信息</a> -->
</div>
```

3. 事件只会触发一次`once`

```javascript
<!-- 事件只触发一次（常用） -->
<button @click.once="showInfo">点我提示信息</button>
```

4. 事件的捕获模式`capture`

> 点击内部元素时事件由外往内触发,默认时是由内往外冒泡

```javascript
<!-- 使用事件的捕获模式,先点击div2 但是触发事件从div1开始触发 -->
<div class="box1" @click.capture="showMsg(1)">
        div1
    <div class="box2" @click="showMsg(2)">
        div2
    </div>
</div>
```

5. 前操作的元素时才触发事件`self`

> 事件发生在哪个标签上,哪个标签就是target,事件冒泡也仍然是源target

```javascript
<!-- 只有event.target是当前操作的元素时才触发事件； -->
<div class="demo1" @click.self="showInfo">
        //点击button事件冒泡到div,但是target仍然是button
        <button @click="showInfo">点我提示信息</button> 
</div>
```

6. 事件的默认行为立即执行，无需等待事件回调执行完毕`passlive`

> 事件回调函数执行完成后再执行事件的默认行为

```javascript
<!-- 事件的默认行为立即执行，无需等待事件回调执行完毕； -->
<ul @wheel.passive="demo" class="list">
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
</ul>
```

### 键盘事件

Vue给常用的一些按键起了别名,一共9个.

- 回车 => enter
- 删除 => delete (捕获“删除”和“退格”键)
- 退出 => esc
- 空格 => space
- 换行 => tab (特殊，必须配合keydown去使用)
- 上 => up
- 下 => down
- 左 => left
- 右 => right

> Vue未提供别名的按键，可以使用按键原始的key值去绑定，但注意要转为kebab-case（短横线命名）

系统修饰键（用法特殊）：ctrl、alt、shift、meta

1. 配合keyup使用：按下修饰键的同时，再按下其他键，随后释放其他键，事件才被触发。
2. 配合keydown使用：正常触发事件。

也可以使用keyCode去指定具体的按键（不推荐）

Vue.config.keyCodes.自定义键名 = 键码，可以去定制按键别名

```javascript
<div id="root">
	<input type="text" placeholder="按下回车提示输入" @keydown.huiche="showInfo">
</div>
<script type="text/javascript">
    Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。
    Vue.config.keyCodes.huiche = 13 //定义了一个别名按键

    new Vue({
        el:'#root',
        data:{
            name:'尚硅谷'
        },
        methods: {
            showInfo(e){
                console.log(e.target.value)
            }
        },
    })
</script>
```

## 计算属性

> Vue在data中存储`属性`,在computed中存储`计算属性`
>
> computed仅在需要计算属性的情况下使用,不包含计算变量等. 属性是属于Vue实例的

1. 定义：要用的属性不存在，要通过已有属性计算得来。
2. .原理：底层借助了Objcet.defineproperty方法提供的getter和setter。
3. get函数什么时候执行？
   1. 初次读取时会执行一次。
   2. 当依赖的数据发生改变时会被再次调用。

4. 优势：与methods实现相比，内部有缓存机制（复用），效率更高，调试方便。
5. 备注：
   1. 计算属性最终会出现在vm上，直接读取使用即可。
   2. 如果计算属性要被修改，那必须写set函数去响应修改，且set中要引起计算时依赖的数据发生改变。

```javascript
<div id="root">
    姓：<input type="text" v-model="firstName"> <br/><br/>
    名：<input type="text" v-model="lastName"> <br/><br/>
  全名：<span>{{fullName}}</span> 
	   <br/><br/>
</div>
<script type="text/javascript">
		const vm = new Vue({
			el:'#root',
			data:{
				firstName:'张',
				lastName:'三',
			},
			methods: {
				demo(){
					
				}
			},
			computed:{
				fullName:{
					//get有什么作用？当有人读取fullName时，get就会被调用，且返回值就作为fullName的值
					//get什么时候调用？1.初次读取fullName时。2.所依赖的数据发生变化时。
					get(){
						console.log('get被调用了')
						//此处的this是vm, firstName lastName都是属于vm对象的属性, fullName也是
						return this.firstName + '-' + this. lastName
					},
					//set什么时候调用? 当fullName被修改时。不需要修改fullname时set可以省略
					set(value){
						const arr = value.split('-')
						this.firstName = arr[0]
						this.lastName = arr[1]
					}
				}
			}
		})
</script>
```

computed简写方法

```javascript
	<script type="text/javascript">
		const vm = new Vue({
			el:'#root',
			data:{
				firstName:'张',
				lastName:'三',
			},
			computed:{
				//完整写法
				/* fullName:{
					get(){
						console.log('get被调用了')
						return this.firstName + '-' + this.lastName
					},
					set(value){
						console.log('set',value)
						const arr = value.split('-')
						this.firstName = arr[0]
						this.lastName = arr[1]
					}
				} */
				//简写
				fullName(){
					console.log('get被调用了')
					return this.firstName + '-' + this.lastName
				}
			}
		})
	</script>
```

## 属性监听

`watch`：

1. 当被监视的属性变化时, 回调函数自动调用, 进行相关操作
2. 监视的属性必须存在，才能进行监视！！
3. 监视的两种写法：
   1. new Vue时传入watch:配置
   2. 通过vm.$watch()监视

### 注意

>computed和watch之间的区别：
>
>- **computed能完成的功能，watch都可以完成。**
>- **watch能完成的功能，computed不一定能完成，例如：watch可以进行异步操作。**
>
>两个重要的小原则：
>
>- **所被Vue管理的函数，最好写成普通函数，这样this的指向才是vm 或 组件实例对象。**
>- **所有不被Vue所管理的函数（定时器的回调函数、ajax的回调函数等、Promise的回调函数），最好写成箭头函数，这样this的指向才是vm 或 组件实例对象。**

### 深度监听

	深度监视：
	(1).Vue中的watch默认不监测对象内部值的改变（一层）。
	(2).配置deep:true可以监测对象内部值改变（多层）。
	备注：
	(1).Vue自身可以监测对象内部值的改变，但Vue提供的watch默认不可以！
	(2).使用watch时根据数据的具体结构，决定是否采用深度监视。

```javascript
	<body>
		<!-- 准备好一个容器-->
		<div id="root">
			<h2>今天天气很{{info}}</h2>
			<button @click="changeWeather">切换天气</button>
		</div>
	</body>

	<script type="text/javascript">
		const vm = new Vue({
			el:'#root',
			data:{
				isHot:true,
			},
			computed:{
				info(){
					return this.isHot ? '炎热' : '凉爽'
				}
			},
			methods: {
				changeWeather(){
					this.isHot = !this.isHot
				}
			},
			watch:{
				//正常写法
				isHot:{
					// immediate:true, //初始化时让handler调用一下
					// deep:true,//深度监视,可以监听属性内部属性的变化 例如 person.name的变化
                    //监听 属性内部某一属性变化 可以 'person.name'(newValue,oldValue){...}
					handler(newValue,oldValue){
						console.log('isHot被修改了',newValue,oldValue)
					}
				}
				//简写
				/* isHot(newValue,oldValue){
					console.log('isHot被修改了',newValue,oldValue,this)
				} */
			}
		})
	</script>
```

方式2

```javascript
vm.$watch('isHot',{
    immediate:true, //初始化时让handler调用一下
    deep:true,//深度监视
    handler(newValue,oldValue){
        console.log('isHot被修改了',newValue,oldValue)
    }
})
//简写
vm.$watch('isHot',(newValue,oldValue)=>{
	console.log('isHot被修改了',newValue,oldValue,this)
})
```



## 绑定样式

使用v-bind: 或者简写:绑定class或者style属性,动态的给定值

绑定样式：

1. class样式
   1. 写法:class="xxx" xxx可以是字符串、对象、数组。
      字符串写法适用于：类名不确定，要动态获取。
      对象写法适用于：要绑定多个样式，个数不确定，名字也不确定。
      数组写法适用于：要绑定多个样式，个数确定，名字也确定，但不确定用不用。
   2. style样式
      :style="{fontSize: xxx}"其中xxx是动态值。
      :style="[a,b]"其中a、b是样式对象。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8" />
		<title>绑定样式</title>
		<style>
			.basic{
				width: 400px;
				height: 100px;
				border: 1px solid black;
			}
			
			.happy{
				border: 4px solid red;;
				background-color: rgba(255, 255, 0, 0.644);
				background: linear-gradient(30deg,yellow,pink,orange,yellow);
			}
			.sad{
				border: 4px dashed rgb(2, 197, 2);
				background-color: gray;
			}
			.normal{
				background-color: skyblue;
			}

			.atguigu1{
				background-color: yellowgreen;
			}
			.atguigu2{
				font-size: 30px;
				text-shadow:2px 2px 10px red;
			}
			.atguigu3{
				border-radius: 20px;
			}
		</style>
		<script type="text/javascript" src="../js/vue.js"></script>
	</head>
	<body>
		<div id="root">
			<!-- 绑定class样式--字符串写法，适用于：样式的类名不确定，需要动态指定 -->
			<div class="basic" :class="mood" @click="changeMood">{{name}}</div> <br/><br/>

			<!-- 绑定class样式--数组写法，适用于：要绑定的样式个数不确定、名字也不确定 -->
			<div class="basic" :class="classArr">{{name}}</div> <br/><br/>

			<!-- 绑定class样式--对象写法，适用于：要绑定的样式个数确定、名字也确定，但要动态决定用不用 -->
			<div class="basic" :class="classObj">{{name}}</div> <br/><br/>

			<!-- 绑定style样式--对象写法 -->
			<div class="basic" :style="styleObj">{{name}}</div> <br/><br/>
			<!-- 绑定style样式--数组写法 -->
			<div class="basic" :style="styleArr">{{name}}</div>
		</div>
	</body>

	<script type="text/javascript">
		Vue.config.productionTip = false
		
		const vm = new Vue({
			el:'#root',
			data:{
				name:'尚硅谷',
				mood:'normal',
				classArr:['atguigu1','atguigu2','atguigu3'],
				classObj:{
					atguigu1:false,
					atguigu2:false,
				},
				styleObj:{
					fontSize: '40px',
					color:'red',
				},
				styleObj2:{
					backgroundColor:'orange'
				},
				styleArr:[
					{
						fontSize: '40px',
						color:'blue',
					},
					{
						backgroundColor:'gray'
					}
				]
			},
			methods: {
				changeMood(){
					const arr = ['happy','sad','normal']
					const index = Math.floor(Math.random()*3)
					this.mood = arr[index]
				}
			},
		})
	</script>
</html>
```

