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

