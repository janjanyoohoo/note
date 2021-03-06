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

## 条件渲染

条件渲染：

1. `v-if`
   写法：
   1. `v-if`="表达式" 
   2. `v-else-if`="表达式"
   3. `v-else`

> 适用于：切换频率较低的场景
>
> 特点：不展示的DOM元素直接被移除。
>
> 注意：v-if可以和:v-else-if、v-else一起使用，但要求结构不能被“打断”。v-else不带条件

2. `v-show`
   写法：v-show="表达式"

> 适用于：切换频率较高的场景。
> 特点：不展示的DOM元素未被移除，仅仅是使用样式隐藏掉

3. 备注：使用v-if的时，元素可能无法获取到，而使用v-show一定可以获取到。

```java
		<div id="root">
			<h2>当前的n值是:{{n}}</h2>
			<button @click="n++">点我n+1</button>
			<!-- 使用v-show做条件渲染 -->
			<!-- <h2 v-show="false">欢迎来到{{name}}</h2> -->
			<!-- <h2 v-show="1 === 1">欢迎来到{{name}}</h2> -->

			<!-- 使用v-if做条件渲染 -->
			<!-- <h2 v-if="false">欢迎来到{{name}}</h2> -->
			<!-- <h2 v-if="1 === 1">欢迎来到{{name}}</h2> -->

			<!-- v-else和v-else-if -->
			<!-- <div v-if="n === 1">Angular</div>
			<div v-else-if="n === 2">React</div>
			<div v-else-if="n === 3">Vue</div>
			<div v-else>哈哈</div> -->

			<!-- v-if与template的配合使用 -->
			<template v-if="n === 1">
				<h2>你好</h2>
				<h2>尚硅谷</h2>
				<h2>北京</h2>
			</template>
		</div>

	<script type="text/javascript">
		const vm = new Vue({
			el:'#root',
			data:{
				name:'尚硅谷',
				n:0
			}
		})
	</script>
```

## 列表

`v-for`遍历指令

1. 用于展示列表数据
2. 语法：`v-for="(item, index) in xxx" :key="yyy"`
   1. key不可以使用index,应该使用唯一的ID,否则渲染会出现混乱
3. 可遍历：数组、对象、字符串（用的很少）、指定次数（用的很少）

---

key的原理

>1. 虚拟DOM中key的作用：
>									key是虚拟DOM对象的标识，当数据发生变化时，Vue会根据【新数据】生成【新的虚拟DOM】, 
>									随后Vue进行【新虚拟DOM】与【旧虚拟DOM】的差异比较，比较规则如下：
>2. 对比规则：
>      								(1).旧虚拟DOM中找到了与新虚拟DOM相同的key：
>           											①.若虚拟DOM中内容没变, 直接使用之前的真实DOM！
>           											②.若虚拟DOM中内容变了, 则生成新的真实DOM，随后替换掉页面中之前的真实DOM。
>3. 用index作为key可能会引发的问题：
>      										1. 若对数据进行：逆序添加、逆序删除等破坏顺序操作:
>           														会产生没有必要的真实DOM更新 ==> 界面效果没问题, 但效率低。
>                 										2. 如果结构中还包含输入类的DOM：
>                      	会产生错误DOM更新 ==> 界面有问题。
>4. 开发中如何选择key?:
>      1. 最好使用每条数据的唯一标识作为key, 比如id、手机号、身份证号、学号等唯一值。
>      2. 如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，
>
>使用index作为key是没有问题的。

### 列表渲染

```javascript
<div id="root">
			<!-- 遍历数组 -->
			<h2>人员列表（遍历数组）</h2>
			<ul>
				<li v-for="(p,index) of persons" :key="index">
					{{p.name}}-{{p.age}}
				</li>
			</ul>

			<!-- 遍历对象 -->
			<h2>汽车信息（遍历对象）</h2>
			<ul>
				<li v-for="(value,k) of car" :key="k">
					{{k}}-{{value}}
				</li>
			</ul>

			<!-- 遍历字符串 -->
			<h2>测试遍历字符串（用得少）</h2>
			<ul>
				<li v-for="(char,index) of str" :key="index">
					{{char}}-{{index}}
				</li>
			</ul>
			
			<!-- 遍历指定次数 -->
			<h2>测试遍历指定次数（用得少）</h2>
			<ul>
				<li v-for="(number,index) of 5" :key="index">
					{{index}}-{{number}}
				</li>
			</ul>
</div>

		<script type="text/javascript">
			new Vue({
				el:'#root',
				data:{
					persons:[
						{id:'001',name:'张三',age:18},
						{id:'002',name:'李四',age:19},
						{id:'003',name:'王五',age:20}
					],
					car:{
						name:'奥迪A8',
						price:'70万',
						color:'黑色'
					},
					str:'hello'
				}
			})
		</script>
```

### 列表过滤

```javascript
//watch实现  (不推荐)
new Vue({
    el:'#root',
    data:{
        keyWord:'',
        persons:[
            {id:'001',name:'马冬梅',age:19,sex:'女'},
            {id:'002',name:'周冬雨',age:20,sex:'女'},
            {id:'003',name:'周杰伦',age:21,sex:'男'},
            {id:'004',name:'温兆伦',age:22,sex:'男'}
        ],
        filPerons:[]
    },
    watch:{
        keyWord:{
            immediate:true,
            handler(val){
                this.filPerons = this.persons.filter((p)=>{
                    return p.name.indexOf(val) !== -1
                })
            }
        }
    }
})  
//用computed实现 (推荐)
new Vue({
    el:'#root',
    data:{
        keyWord:'',
        persons:[
            {id:'001',name:'马冬梅',age:19,sex:'女'},
            {id:'002',name:'周冬雨',age:20,sex:'女'},
            {id:'003',name:'周杰伦',age:21,sex:'男'},
            {id:'004',name:'温兆伦',age:22,sex:'男'}
        ]
    },
    computed:{
        filPerons(){
            return this.persons.filter((p)=>{
                return p.name.indexOf(this.keyWord) !== -1
            })
        }
    }
}) 
```

### 列表排序

```javascript
<ul>
    <li v-for="(p,index) of filPerons" :key="p.id">
        {{p.name}}-{{p.age}}-{{p.sex}}
    	<input type="text">
	</li>
</ul>
new Vue({
    el:'#root',
    data:{
        keyWord:'',
        sortType:0, //0原顺序 1降序 2升序
        persons:[
            {id:'001',name:'马冬梅',age:30,sex:'女'},
            {id:'002',name:'周冬雨',age:31,sex:'女'},
            {id:'003',name:'周杰伦',age:18,sex:'男'},
            {id:'004',name:'温兆伦',age:19,sex:'男'}
        ]
    },
    computed:{
        filPerons(){
            const arr = this.persons.filter((p)=>{
                return p.name.indexOf(this.keyWord) !== -1
            })
            //判断一下是否需要排序
            if(this.sortType){
                arr.sort((p1,p2)=>{
                    return this.sortType === 1 ? p2.age-p1.age : p1.age-p2.age
                })
            }
            return arr
        }
    }
}) 
```

## 监测数据

Vue监视数据的原理：
   1. vue会监视`data中所有层次的数据`。

   2. 如何监测对象中的数据？

         				1. 通过`setter`实现监视，且要在new Vue时就传入要监测的数据。

      - **对象中后追加的属性，Vue默认不做响应式处理**
      - 如需给后添加的属性做响应式，请使用如下API：
        - `Vue.set(target，propertyName/index，value) `
        - `vm.$set(target，propertyName/index，value)`

   3. 如何监测数组中的数据？

         				1. 通过**包裹数组更新元素的方法实现**，本质就是做了两件事：
                - 调用原生对应的方法对数组进行更新。
                - 重新解析模板，进而更新页面。

   4. 在Vue修改数组中的某个元素一定要用如下方法：

         				1. 使用这些API:`push()`、`pop()`、`shift()`、`unshift()`、`splice()`、`sort()`、`reverse()`
         				2. `Vue.set() 或 vm.$set()`
                			

> 特别注意：`Vue.set()` 和 `vm.$set() `**不能给vm 或 vm的根数据对象 添加属性！！！**

```javascript
Vue.set(this.student,'sex','男')
this.$set(this.student,'sex','男')
```

## 收集表单数据

1. `<input type="text"/>`，则v-model收集的是value值，用户输入的就是value值。
2. `<input type="radio"/>`，则v-model收集的是value值，且要给标签配置value值。
3. `<input type="checkbox"/>`
   - 没有配置input的value属性，那么收集的就是checked（勾选 or 未勾选，是布尔值）
   - 配置input的value属性:
     - -model的初始值是非数组，那么收集的就是checked（勾选 or 未勾选，是布尔值）
     - v-model的初始值是数组，那么收集的的就是value组成的数组

> `v-model`的三个修饰符：
>
> - `lazy`：失去焦点再收集数据
> - `number`：输入字符串转为有效的数字
> - `trim`：输入首尾空格过滤

## 过滤器

> 对要显示的数据进行特定格式化后再显示（适用于一些简单逻辑的处理）。

1. 注册过滤器：`Vue.filter(name,callback)` 或 `new Vue{filters:{}}`

2. 使用过滤器：`{{ xxx | 过滤器名}} ` 或  `v-bind:属性 = "xxx | 过滤器名"`

   > 过滤器也可以接收额外参数、多个过滤器也可以串联
   > 并没有改变原本的数据, 是产生新的对应的数据

## 内置指令

### v-once

v-once所在节点在初次动态渲染后，就视为静态内容了。

以后数据的改变不会引起v-once所在结构的更新，可以用于优化性能。

### v-cloak

> 没有值

本质是一个特殊属性，Vue实例创建完毕并接管容器后，会删掉v-cloak属性。
使用css配合v-cloak可以解决网速慢时页面展示出{{xxx}}的问题。

### v-html

作用：向指定节点中渲染包含html结构的内容。

与插值语法的区别：

- v-html会替换掉节点中所有的内容，{{xx}}则不会。
- v-html可以识别html结构。

**严重注意：v-html有安全性问题！**

- **在网站上动态渲染任意HTML是非常危险的，容易导致XSS攻击。**
- **一定要在可信的内容上使用v-html，永不要用在用户提交的内容上！**

### v-text

作用：向其所在的节点中渲染文本内容。

与插值语法的区别：v-text会替换掉节点中的内容，{{xx}}则不会。

### v-pre

跳过其所在节点的编译过程。

可利用它跳过：没有使用指令语法、没有使用插值语法的节点，会加快编译。

### 其他指令

```txt
v-bind	: 单向绑定解析表达式, 可简写为 :xxx
v-model	: 双向数据绑定
v-for  	: 遍历数组/对象/字符串
v-on   	: 绑定事件监听, 可简写为@
v-if 	 	: 条件渲染（动态控制节点是否存存在）
v-else 	: 条件渲染（动态控制节点是否存存在）
v-show 	: 条件渲染 (动态控制节点是否展示)
```

## 自定义指令

1. 定义语法：
   - 局部指令：

```javascript
		new Vue({
			el:'#root',
			data:{
				name:'尚硅谷',
				n:1
			},
			directives:{
				//big函数何时会被调用？1.指令与元素成功绑定时（一上来）。2.指令所在的模板被重新解析时。
				/* 'big-number'(element,binding){
					// console.log('big')
					element.innerText = binding.value * 10
				}, */
				big(element,binding){
					console.log('big',this) //注意此处的this是window
					// console.log('big')
					element.innerText = binding.value * 10
				},
				fbind:{
					//指令与元素成功绑定时（一上来）
					bind(element,binding){
						element.value = binding.value
					},
					//指令所在元素被插入页面时
					inserted(element,binding){
						element.focus()
					},
					//指令所在的模板被重新解析时
					update(element,binding){
						element.value = binding.value
					}
				}
			}
		})
```

- 全局指令：

```javascript
//定义全局指令
 Vue.directive('fbind',{
     //指令与元素成功绑定时（一上来）
     bind(element,binding){
         element.value = binding.value
     },
     //指令所在元素被插入页面时
     inserted(element,binding){
         element.focus()
     },
     //指令所在的模板被重新解析时
     update(element,binding){
         element.value = binding.value
     }
 }) 
```

2. 配置对象中常用的3个回调：
   - **bind**：指令与元素成功绑定时调用。
   - **inserted**：指令所在元素被插入页面时调用。
   - **update**：指令所在模板结构被重新解析时调用。

3. 备注：
   - **指令定义时不加v-，但使用时要加v-**
   - **指令名如果是多个单词，要使用kebab-case命名方式，不要用camelCase命名。**

## 生命周期

> 生命周期回调函数、生命周期函数、生命周期钩子。
>
> 共有4个节点,8个钩子

是什么：

- Vue在关键时刻帮我们调用的一些特殊名称的函数。

生命周期函数的名字不可更改，但函数的具体内容是程序员根据需求编写的。

生命周期函数中的this指向是vm 或 组件实例对象。

---

常用的生命周期钩子：

- **created**: 网络请求初始化页面

- **mounted**: 发送ajax请求、启动定时器、绑定自定义事件、订阅消息等【初始化操作】。
- **beforeDestroy**: 清除定时器、解绑自定义事件、取消订阅消息等【收尾工作】。
  - 释放资源

关于销毁Vue实例

- 销毁后借助Vue开发者工具看不到任何信息。
- 销毁后自定义事件会失效，但原生DOM事件依然有效。
- 一般不会在beforeDestroy操作数据，因为即便操作数据，也不会再触发更新流程了。

![image-20210927230435706](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210927230442.png)

```javascript
new Vue({
			el:'#root',
			// template:`
			// 	<div>
			// 		<h2>当前的n值是：{{n}}</h2>
			// 		<button @click="add">点我n+1</button>
			// 	</div>
			// `,
			data:{
				n:1
			},
			methods: {
				add(){
					console.log('add')
					this.n++
				},
				bye(){
					console.log('bye')
					this.$destroy()
				}
			},
			watch:{
				n(){
					console.log('n变了')
				}
			},
			beforeCreate() {
				console.log('beforeCreate')
			},
			created() {
				console.log('created')
			},
			beforeMount() {
				console.log('beforeMount')
			},
			mounted() {
				console.log('mounted')
			},
			beforeUpdate() {
				console.log('beforeUpdate')
			},
			updated() {
				console.log('updated')
			},
			beforeDestroy() {
				console.log('beforeDestroy')
			},
			destroyed() {
				console.log('destroyed')
			},
		})
```

# 进阶

## 多文件组件

### 使用组件的三大步骤
- 定义组件(创建组件)
- 注册组件
- 使用组件(写组件标签)

----

1. 如何定义一个组件？

   - 使用Vue.extend(options)创建，其中options和new Vue(options)时传入的那个options几乎一样，但也有点区别；

     - **.el不要写**

       - 为什么？ ——— 最终所有的组件都要经过一个vm的管理，由vm中的el决定服务哪个容器。

     - **data必须写成函数**

       - 为什么？ ———— 避免组件被复用时，数据存在引用关系。(指向同一个对象,函数则是每次返回一个新对象)

         > 备注：使用template可以配置组件结构。

2. 如何注册组件？
   - 局部注册：靠new Vue的时候传入components选项
   - 全局注册：靠`Vue.component('组件名',组件)`

3. 编写组件标签：

```html
<html>
	<body>
		<div id="root">
			<hello></hello>
			<hr>
			<h1>{{msg}}</h1>
			<hr>
			<!-- 第三步：编写组件标签 -->
			<school></school>
			<hr>
			<!-- 第三步：编写组件标签 -->
			<student></student>
		</div>

		<div id="root2">
			<hello></hello>
		</div>
	</body>
	<script type="text/javascript">
		//第一步：创建school组件
		const school = Vue.extend({
			template:`
				<div class="demo">
					<h2>学校名称：{{schoolName}}</h2>
					<h2>学校地址：{{address}}</h2>
					<button @click="showName">点我提示学校名</button>	
				</div>
			`,
			// el:'#root', //组件定义时，一定不要写el配置项，因为最终所有的组件都要被一个vm管理，由vm决定服务于哪个容器。
			data(){
				return {
					schoolName:'尚硅谷',
					address:'北京昌平'
				}
			},
			methods: {
				showName(){
					alert(this.schoolName)
				}
			},
		})

		//第一步：创建student组件
		const student = Vue.extend({
			template:`
				<div>
					<h2>学生姓名：{{studentName}}</h2>
					<h2>学生年龄：{{age}}</h2>
				</div>
			`,
			data(){
				return {
					studentName:'张三',
					age:18
				}
			}
		})
		
		//第一步：创建hello组件
		const hello = Vue.extend({
			template:`
				<div>	
					<h2>你好啊！{{name}}</h2>
				</div>
			`,
			data(){
				return {
					name:'Tom'
				}
			}
		})
		
		//第二步：全局注册组件
		Vue.component('hello',hello)

		//创建vm
		new Vue({
			el:'#root',
			data:{
				msg:'你好啊！'
			},
			//第二步：注册组件（局部注册）
			components:{
				school,
				student
			}
		})
		new Vue({
			el:'#root2',
		})
	</script>
</html>
```

### 注意点

1. 关于组件名:

   - 一个单词组成：

     - 第一种写法(首字母小写)：school
     - 第二种写法(首字母大写)：School
     - 多个单词组成：
       - 第一种写法(kebab-case命名)：`my-school` (个人推荐)
       - 第二种写法(CamelCase命名)：`MySchool` (需要Vue脚手架支持)

   - 备注：

     > (1).组件名尽可能回避HTML中已有的元素名称，例如：h2、H2都不行。
     > (2).可以使用name配置项指定组件在开发者工具中呈现的名字。

2. 关于组件标签:

   - 第一种写法：<school></school>

   - 第二种写法：<school/>

     > 备注：不用使用脚手架时，<school/>会导致后续组件不能渲染。

3. 一个简写方式：
   - `const school = Vue.extend(options)` 可简写为：`const school = options`





