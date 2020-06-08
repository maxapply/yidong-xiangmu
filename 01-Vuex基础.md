# Vuex



## 组件之间传值回看

### 兄弟之间

![1573222656185](img(online)/1573222656185.png)

实现步骤：

1. 定义模块 src/bus.js(公共事件对象)，内容就是**导入**Vue模块并**导出**一个Vue实例对象

   ```js
   import Vue from 'vue'
   export default new Vue() 
   ```

   

2. 在**各个兄弟**组件中，导入 bus.js 模块

   ```js
   import bus from '@/bus.js'
   ```

   > 虽然bus.js被各个组件都导入，但是系统中bus只有一份

3. 在**接收**数据的兄弟组件的 created 生命周期方法里(使得事件及时响应)，"大哥"的组件中声明

   使用 bus.$on('事件名称', (接收的数据) => {}) 定义事件成员方法

   ```js
   created(){
     // 定义事件,注意箭头函数应用
     bus.$on('xxx', data=>{
       console.log(data)
     })
   }
   ```

   > xxx是事件方法的名称
   >
   > data是形参，待接收数据，并且可以定义多个
   >
   > 注意：如果$on内部要使用this，请通过"**箭头函数**"声明方法

4. 在**发送**数据的兄弟组件中，使用 bus.$emit('事件名称', 要发送的数据) 来向外发送数据

   ```vue
   <button @click="sendMsg">给兄弟说话</button>
   
   <script>
   export default {
     methods: {
       sendMsg(){
         // 触发 绑定的 事件，并向外传递参数
         bus.$emit('xxx', '1000元保护费')
       }
     }
}
   </script>
   ```
   
   > xxx 是接收数据组件给bus声明的方法





### 父给子

父组件要在子组件标签上通过**属性值**方式传值

```html
<子组件标签 name=value name=value name=value></子组件标签>
```

子组件接收并应用父传递来的数据，具体通过props接收

```html
<!--在模板中应用传递来的数据-->
<input :style="{color:xx}">

<script>
  export default {
    // 通过props接收父传递过来的数据,注意各个名称需要使用单引号圈选
    // 1. 简单接收
    props:['xx','xx','xx'],
    // 2. 复杂接收
    props:{
      xx:{
        type:类型限制，String、Number、Array
        default:默认值
      }
    }
  }
</script>
```



### 子给父

**`父组件操作语法`**：

父组件通过**@符号**向子组件传递一个<font color=red>事件方法</font>

```
<子组件 @func1="show"></子组件>
...
methods:{
	show(arg1,arg2){xxxx}
}
```

其中

​	func1为事件的名称，给子组件触发使用

​	show为该事件的执行函数，在父组件内部的methods中定义好

​	在事件中可以通过形参(arg1、arg2)接收子组件出来过来的数据 并做近一步处理



**`子组件操作`**：

子组件中，使用<font color=red>this.$emit()</font>调用 父组件中的方法

```js
this.$emit('func1', 数据, 数据)
this:当前子组件对象
$emit:通过这个关键字可以使得子组件可以调用 自己 的事件(父组件给声明的)
```

从第二个位置开始传递实参数据，其可以被父组件methods方法的形参所接受

$emit(名称，数据，数据……) 是组件调用自己方法的固定方式，第一个参数是被调用方法的名称，后续参数都是给方法传递的实参信息



## 什么是Vuex

官网：https://vuex.vuejs.org/zh/



Vuex 是一个专为 Vue.js 应用程序开发的  数据状态管理模式。

它采用集中式存储管理应用中  所有组件的共享数据



**Vuex是组件之间数据共享的一种机制**

![1581065897149](img(online)/1581065897149.png)

 

## 为什么要有Vuex

父子传值或兄弟传值，太麻烦了，不好管理

vuex提供了一种全新的数据共享管理模式，该模式相比较简单的组件传值更加高端、大气、上档次。



`注意`：

1. 只有组件间共享的数据，才有必要存储到vuex中，组件自己私有的数据，还是要存储到自己的data中；

2. 在应用的时候要因地制宜，普通组件传值  和  vuex 适合应用在不同场合，要注意区分

 

# 初始化

## 初始化项目

创建一个新vuecli 项目： vue  create  01-pro

![1581211435907](img(online)/1581211435907.png)

> Vuex暂时不要勾选



`配置`：

1. 给项目做配置vue.config.js

   ```js
   module.exports = {
     lintOnSave: false,
     devServer: {
       open: true
     }
   }
   
   ```

2. 删除components/HelloWorld.vue文件

3. 初始化App.vue文件，内容如下：

   ```vue
   <template>
       <div>
         <h2>App根基组件</h2>
       </div>
   </template>
   
   <script>
   export default {
     name: 'App'
   }
   </script>
   
   <style lang="less" scoped>
   </style>
   
   ```

4. 执行命名： npm run serve 启动项目



`效果`：

![1580612250804](img(online)/1580612250804.png)

  

## 创建应用组件

`步骤`：

1. 在src/components 下创建两个应用组件**First.vue** 和 **Second.vue**，并填充如下内容

   First.vue

   ```vue
   <template>
       <div id="first">
         <p>我是大哥组件</p>
       </div>
   </template>
   
   <script>
   export default {
     name: 'first'
   }
   </script>
   
   <style lang="less" scoped>
   #first{
     width: 350px;
     height: 200px;
     border:2px solid blue;
   }
   </style>
   
   ```

   Second.vue

   ```vue
   <template>
       <div id="second">
         <p>我是小弟组件</p>
       </div>
   </template>
   
   <script>
   export default {
     name: 'second'
   }
   </script>
   
   <style lang="less" scoped>
   #second{
     width: 350px;
     height: 200px;
     border:2px solid green;
     margin-top:10px;
   }
   </style>
   
   ```

2. App.vue对First.vue 和 Second.vue 做  导入、注册、使用

   ```vue
   <template>
       <div id="app">
         <h2>App根组件-Vuex</h2>
         <!--使用-->
         <first></first>
         <second></second>
       </div>
   </template>
   
   <script>
   // 导入
   import First from './components/First'
   import Second from './components/Second'
   export default {
     name: 'app',
     // 注册
     components: {
       First,
       Second
     }
   }
   </script>
   
   <style lang="less" scoped>
   #app{
     width: 400px;
     height: 600px;
     border:2px solid red;
   }
   </style>
   
   ```



访问效果：

![1581060679725](img(online)/1581060679725.png)



## 安装配置vuex

1. 安装vuex

   ```bash
   npm i vuex
   ```

   

2. main.js做如下设置，4步走

   ```js
   import Vue from 'vue'
   import App from './App.vue'
   
   // 1. 导入vuex模块
   import Vuex from 'vuex'
   
   // 2. 对Vuex进行注册，插件机制
   //    本质是要给Vue创建全局成员
   //    Vue.prototype.$store = xxx
   //    后续，在业务组件中就可以访问$store了
   Vue.use(Vuex)
   
   // 3. 创建Vuex对象，Vuex.Store整体才是"构造函数"
   //    对象成员名称store，是自定义的，考虑到后边代码的兼容性，就用store
   /*
       const Vuex = {
         Store:function(){}
       }
   */
   const store = new Vuex.Store({
     // 配置共享数据,"state"就是固定配置数据的名称，不能自定义
     state: {
       // 数据成员，名称可以自定义，数量不限制
       count: 25,
       count1: 26
       // ……
     }
   })
   
   Vue.config.productionTip = false
   
   new Vue({
     // 4. 挂载Vuex对象,Vue内部还可以填充名称为store的成员
     store, // 简易成员赋值，完整：store:store
     render: h => h(App)
   }).$mount('#app')
   
   ```
   



# 使用Vuex

## 访问

### state[记住]

`目标`：

​	在各个组件中访问vuex共享的数据

`语法`：

```js
this.$store.state.xxx   	// 组件内
$store.state.xxx 				// 模板中，没有this关键字
```

> xxx：代表共享数据的名称



`操作`：

给First.vue设置如下代码

```vue
<template>
    <div id="first">
      <p>我是大哥组件</p>
      <p>count的值为：{{$store.state.count}}</p>
    </div>
</template>
```



给Second.vue 设置如下代码：

```vue
<template>
    <div id="second">
      <p>我是小弟组件</p>
      <p>count的值为：{{$store.state.count}}</p>
    </div>
</template>
```



`效果`：

![1581060982467](img(online)/1581060982467.png)

`注意`：

​	vuex数据既可以在**模板中**被访问，语法 **$store.state.xxx**

​	也可以在**组件实例**内部被访问，语法 **this.\$store.state.xxx**

 

### mapState

`目标`：

​	掌握  mapState  方式访问state成员



`引言`：

​	组件中的Vuex数据如果需要**频繁**被访问，那么类似这样的代码  this.$store.state.xxx 或 \$store.state.xxx 需要被重复编写，显然，该代码过于复杂，会增加很多工作量，有没有简便的方式处理呢? 

答：可以使用**mapState**，其可以使得各个state成员像组件实例**计算属性成员**一样被直接访问



语法：

1. 在应用组件里边   按需导入 mapState 辅助函数

   ```js
   import { mapState } from "vuex";
   ```

   

2. 定义计算属性

   ```js
   computed: {
   	// 通过 展开运算符，把 state 中的数据成员xx、yy、zz映射为计算属性成员
     ...mapState([xx,yy,zz……])
   }
   
   ```
   
   > 现在在组件中就可以像访问**计算属性**一样访问Vuex成员了
   



使用示例：

在Second.vue组件中通过mapState方式设置并访问共享数据：

```vue
<template>
    <div id="second">
      <p>我是小弟组件</p>
      <p>count的值为：
        <!--类似访问普通  计算属性  成员一样，访问vuex数据-->
        {{count}}--
        {{count}}--
        {{count1}}--
        {{count1}}--
      </p>
    </div>
</template>

<script>
// 通过mapState对共享数据做简化获取处理
import { mapState } from 'vuex'
export default {
  name: 'second',
  // 计算属性
  computed: {
    // 处理mapState，三个点是做展开运算的，运算结果与后边的内容是一致的
    ...mapState(['count', 'count1'])
  }
}
</script>
```



`效果`：

![1581216183855](img(online)/1581216183855.png)



`注意`：

1. mapState是把共享数据配置为**computed计算属性**的成员
2. 成员越多，mapState优势越明显

 

### mapState原理

了解mapState实现原理

```js
computed: {
	// 通过 展开运算符，把 state 中的数据成员xx、yy、zz映射为计算属性成员
  ...mapState(  [xx,yy,zz……]  )
}
/**
mapState函数本质可以理解为如下：
function mapState(arr){
  return {
    arr[0]:function(){
      return this.$store.state.arr[0]
    },
    arr[1]:function(){
      return this.$store.state.arr[1]
    }
    ……
  }
}
*/

// 故有如下结论：
computed: {
  ...mapState(['count', 'count1'])
}
// 等同于下述结果
computed: {
  count: function () {
    return this.$store.state.count
  },
  count1: function () {
    return this.$store.state.count1
  }
}
// 即mod可以当做methods方法被直接访问
```



## 修改(同步)

修改store中state上的值，需要借助mutations

### mutations[记住]

`目标`：

​	通过mutations对state共享数据进行修改



`声明语法`：

```js
// 在实例化Vuex对象的参数对象中，定义mutations成员(与state平级)对象
mutations:{
 	// 参数state是固定的，代表vuex本身的state(名称可以自定义，为了可读性就用state即可)，
  // 可以用以获取其中的共享数据
  // 第二个参数可以接收 应用层 参数信息
	方法名称(state,形参){
		通过state修改内部的数据
	}
}
```

 

`调用 mutations 语法`：

```js
// 组件实例调用
this.$store.commit('mutations方法名')
this.$store.commit('mutations方法名',实参)
// 模板中调用
$store.commit('mutations方法名')  
$store.commit('mutations方法名',实参)
```



应用示例：

1. 在main.js中给vuex声明mutations和相关成员

   ```js
   // 设置共享数据"修改"机制
   // 业务组件不能直接修改数据，具体要通过vuex进行
   // mutations是固定成员名称，不能自定义，代表对共享数据做修改的
   mutations: {
     // 内部可以声明许多"方法"，针对不同的共享数据做修改的
     // 【state】固定参数，名字可以自定义，但是我们就用这个名字，
     // 可读性比较好，就代表共享数据的"state对象"，因此可以访问共享数据
     // 【data】是形参，代表被修改数据传递的参数，名字可以自定义，用于接收应用层传递过来的数据
     // (mod(10)=======>count=35)
     mod: function (state, data) {
       // 对count做修改
       state.count += data
     }
   }
   ```

   

2. 在First.vue中调用mutations成员

   ```vue
   <template>
     <div id="first">
       <p>我是大哥组件</p>
       <p>count的值为：{{$store.state.count}}</p>
       <p>
         <button @click="$store.commit('mod', 10)">修改count</button>
         <button @click="$store.commit('mod', 20)">修改count</button>
         <button @click="$store.commit('mod', 30)">修改count</button>
       </p>
     </div>
   </template>
   ```
   
> 在组件实例内部 可以 通过this调用，例如 this.$store.commit('mod',10)
   >
   > 调用一次或多次都可以



`效果`：

![1581217106116](img(online)/1581217106116.png)

`注意`：

1. 不要对**this.$store.state.xxx=yyy** 直接修改，容易造成数据混乱，因为许多组件都可以这样干

2. 上图各个应用的地方都有变化，可以知道vuex有**响应式**效果



### mapMutations

组件中的Vuex数据如果需要频繁被修改，那么类似这样的代码 this.$store.commit(xxx,yyy) 需要被重复编写，显然，该代码有点复杂，会增加很多工作量，有没有简便的方式处理呢? 

答：可以使用**mapMutations**，其可以使得各个mutations成员像methods方法一样直接被访问



`使用步骤`：

1. 在应用组件中做mapMutations 的模块化导入操作

   ```js
   import { mapMutations } from "vuex";
   ```

   

2. 在methods方法中做如下设置

   ```js
   methods: {
     ...mapMutations([xx,yy,zz……])
   }
   
   ```

   > 注意：
   >
   > ​	mapState 是在**computed**中做展开
   >
   > ​	mapMutations是在**methods**中做展开
   >
   > ​	xx/yy/zz 是mutations成员名称，一个或多个都可以

   


应用示例：

在Second.vue中做如下配置：

```vue
<template>
  <div id="second">
    <p>我是小弟组件</p>
    <p>
      count的值为：
      <!--类似访问普通data成员一样，访问vuex数据-->
      {{count}}--
      {{count}}--
      {{count1}}--
      {{count1}}--
    </p>
    <p>
      <!--mod使用感觉与访问自己的methods方法一致-->
      <button @click="mod(10)">修改count</button>
      <button @click="mod(20)">修改count</button>
      <button @click="mod(30)">修改count</button>
    </p>
  </div>
</template>

<script>
// 导入 mapMutations
import { mapState, mapMutations } from 'vuex'
export default {
  name: 'second',
  computed: {
    ...mapState(['count', 'count1'])
  },
  methods: {
    // 应用mapMutations
    ...mapMutations(['mod'])
  }
}
</script>
```

> 现在可以像访问**methods方法**一样直接访问mutations成员，非常方便



执行效果：

![1581218819304](img(online)/1581218819304.png)

注意：

成员越多、访问越频繁，mapMutations优势越明显



### mapMutations原理

了解mapMutations原理：

```js
methods: {
  ...mapMutations([xx,yy,zz……])
}
/**
mapMutations 函数本质为如下：

function mapMutations(arr){
  return {
    arr[0]:function(arg){
      return this.$store.commit(arr[0],arg)
    },
    arr[1]:function(arg){
      return this.$store.commit(arr[1],arg)
    }
    ……
  }
}
*/

// 故有如下结论：
methods: {
  ...mapMutations(['mod'])
}
// 等同于下述结果
methods:{
  mod (arg) {
    this.$store.commit('mod', arg)
  }
}
```



## 修改(异步)

mutations 和 actions 两个项目都是对Vuex的共享数据最修改的它们的区别是：

mutations用在同步数据修改上，简单理解为页面上有数据要变化就使用mutations

actions用在异步数据修改操作上，简单理解为axios调用服务器端修改数据就使用actions



什么地方有异步操作：

1. ajax/axios
2. setTimeout
3. fs.readFile()
4. ……



`声明语法`：

```js
// actions:固定标志，对共享数据做异步修改
actions: {
  // 声明修改数据的成员方法，数量不限制
  // 参数context：可以自定义，但是就用context(官方就用之)，是一个对象，
  // 具体代表$store(它们不是同一个对象)
  // 参数data：可以自定义名称，用于可以接收 应用层 的数据信息
  成员名称: function (context, data) {
    // actions规定要通过调用mutations成员修改数据(vuex内部规定的)
    context.commit(mutations成员，实参)
  }
}
```



`调用语法`：

```js
this.$store.dispatch('xx',参数)  // 组件内部
$store.dispatch('xx',参数)  // 模板
```

> xx:代表actions内部成员名称



### actions

`目标`：

​	通过actions实现异步方式修改vuex的数据信息



`步骤`：

1. 在vuex中通过actions声明成员，实现异步修改数据操作

   ```js
   // 数据修改--异步方式
   actions: {
     // 修改共享数据的各个方法，数量不限制
     // 【context】是一个形式参数，名称可以自定义，但是我们就用context即可
     //  其代表"$store"对象，因此context可以调用commit方法(当然也可以调用state成员)
     //  【data】形式参数，可以接收应用组件传递来的信息，名字可以自定义
     xiu: function (context, data) {
       // 【异步修改数据要通过commit调用mutations间接修改的】
       // setTimeout() 1秒中之后做修改
       setTimeout(() => {
         context.commit('mod', data)
       }, 1000)
     }
   }
   ```

   > 请求过来后停顿1秒后再操作，体现异步

   

2. First.vue中调用actions方法

   ```vue
   <p>
     <button @click="$store.dispatch('xiu',10)">异步修改count</button>
     <button @click="$store.dispatch('xiu',20)">异步修改count</button>
     <button @click="$store.dispatch('xiu',30)">异步修改count</button>
   </p>
   ```



`效果`：

![1581219934577](img(online)/1581219934577.png)



注意：

​	actions成员内部 既可以操作mutations成员，也可以操作 state成员

​	一般不要操作 state，所有数据修改都通过mutations进行，尽管这样有点繁琐，

​	但是好处是数据管理比较集中、专业，体现了只有mutations才拥有修改数据的**特权**。



### mapActions

组件中的Vuex数据需要频繁被修改，那么类似这样的代码 this.$store.dispatch(xxx) 需要被重复编写，显然，这个代码有点过于复杂，会增加很多工作量，有没有简便的方式处理呢? 

答：可以使用mapActions，其可以使得各个actions方法像methods方法一样直接被访问

 

`使用步骤`：

1. 在应用组件中做mapActions 的模块化导入操作

   ```js
     import { mapActions } from "vuex";
   ```

   

2. 在methods方法中做如下设置

   ```js
   import { mapActions } from "vuex";
   methods: {
   	...mapActions([xx,yy,zz])
   }
   ```
   
   > 注意：
   >
   > ​	mapActions是在methods中做展开
   >
   > ​	xx/yy/zz 是actions成员名称，一个或多个都可以
   



应用示例：

Second.vue中设置如下代码：

```vue
<template>
  <div id="second">
    <p>我是小弟组件</p>
    <p>
      count的值为：
      <!--类似访问普通data成员一样，访问vuex数据-->
      {{count}}--
      {{count}}--
      {{count1}}--
      {{count1}}--
    </p>
    <p>
      <button @click="mod(10)">修改count</button>
      <button @click="mod(20)">修改count</button>
      <button @click="mod(30)">修改count</button>
    </p>
    <p>
      <!-- 异步方式修改数据
          用简便方式实现，像调用methods方法一样，去调用actions成员
      -->
      <button @click="xiu(10)">修改(异步)count</button>
      <button @click="xiu(20)">修改(异步)count</button>
      <button @click="xiu(30)">修改(异步)count</button>
    </p>
  </div>
</template>

<script>
// 导入 mapActions
import { mapState, mapMutations, mapActions } from 'vuex'
export default {
  name: 'second',
  computed: {
    ...mapState(['count', 'count1'])
  },
  methods: {
    ...mapMutations(['mod']),
		// 应用 mapActions
    ...mapActions(['xiu'])
  }
}
</script>
```

 

`效果`：

![1581220674246](img(online)/1581220674246.png)

注意：

成员越多、访问越频繁，mapActions优势越明显



### mapActions原理

了解mapActions原理

```js
methods: {
	...mapActions([xx,yy,zz])
}
/**
mapActions 函数本质为如下：
function mapActions(arr){
  return {
    arr[0]:function(arg){
      return this.$store.dispatch(arr[0],arg)
    },
    arr[1]:function(arg){
      return this.$store.dispatch(arr[1],arg)
    }
    ……
  }
}
*/

// 故有如下结论：
methods:{
  // actions成员越多，越能体现mapActions的优势
  ...mapActions(['xiu'])
}

// 等同于下述结果
methods:{
  xiu (arg) {
    this.$store.dispatch('xiu', arg)
  }
}
```



### 扩展说明

`注意`：

​	mutations也可以实现**异步方式**操作数据，为什么不这样做呢

 	1. devtools调试工具有延迟，造成调试有误差
 	2. 比较混乱，不成规矩
 	3. Vuex倡导者也不推荐这样干



![1581164488416](img(online)/1581164488416.png)

上图来自官网，说明的事情有如下：

1. Actions拥有与后端接口直接交互的特权，一般都是**异步**操作
2. Actions对mutations进行调用，借用 commit
3. mutations的操作可以直观反映在devtools调试工具中，方便程序调试，可以进行页面数据的同步更新上
4. mutations对state进行直接操作
5. state状态数据可以渲染给组件显示
6. 组件实例通过 dispatch 调用actions



# 案例应用

`目标`：

​	通过vuex实现对账户信息的管理维护



## 介绍

案例描述：

​	一般后台项目应用中，用户输入 用户名、密码 登录系统，此时服务器端做账号有效性校验，并使得用户进入后台系统，后台系统中有许多组件对用户的信息都要进行操作，有的组件要显示用户的相关信息，有的组件要判断用户信息是否存在并做不同处理，有的组件要删除用户的信息，因此用户在进入系统的同时还要通过localStorage存储账号信息，以便各组件获取使用。



## 实操

### 创建vuecli项目

![1581130307882](img(online)/1581130307882.png)

路由、Vuex都默认安装

![1583043233258](img(online)/1583043233258.png)

### 项目结构文件初始化

1. 给项目做配置

   vue.config.js

   ```js
   module.exports = {
     lintOnSave: false,
     devServer: {
       open: true
     }
   }
   
   ```

2. 删除views/About.vue文件

3. 删除router/index.js 中About对应的路由

4. 创建views/Login.vue文件、并创建对应路由

   组件内容：

   ```vue
   <template>
       <div>
         <h2>管理员登录</h2>
       </div>
   </template>
   
   <script>
   export default {
     name: 'login'
   }
   </script>
   
   <style lang="less" scoped>
   </style>
   
   ```

   路由：

   ```js
   {
     path: '/login',
     name: 'login',
     component: () => import('@/views/Login.vue')
   }
   ```

5. 初始化views/Home.vue 内容

   ```vue
   <template>
       <div>
         <h2>后台首页</h2>
       </div>
   </template>
   
   <script>
   export default {
     name: 'home'
   }
   </script>
   
   <style lang="less" scoped>
   </style>
   
   ```

   

6. 初始化App.vue文件，内容如下：

   ```vue
   <template>
       <div id="app">
         <!--路由组件占位符-->
         <router-view></router-view>
       </div>
   </template>
   
   <script>
   export default {
     name: 'app'
   }
   </script>
   
   <style lang="less" scoped>
   </style>
   
   ```

7. 执行命名： npm run serve 启动项目



![1583043709962](img(online)/1583043709962.png)

### 绘制登录页面完成登录功能

在views/Login.vue 中做如下设置

```vue
<template>
  <div>
    <h2>管理员登录</h2>
    <!--.trim是修饰符，会自动去除收集到内容的首尾 空格-->
    <p>用户名：<input type="text" v-model.trim="loginForm.username"></p>
    <p>密码：<input type="password" v-model.trim="loginForm.userpass"></p>
    <p><button @click="login()">登录</button></p>
  </div>
</template>

<script>
export default {
  name: 'login',
  data () {
    return {
      loginForm: {
        username: '', // 用户名
        userpass: '' // 密码
      }
    }
  },
  methods: {
    login () {
      // 用户名、密码 非空即可
      if (this.loginForm.username && this.loginForm.userpass) {
        this.$router.push({ name: 'home' })
        // this.$router.push('/')
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>

```

> v-model.trim  ：修饰器，使得获得内容的左右空格自动清除
>
> ​							相关说明  https://vue.docschina.org/v2/guide/forms.html#trim 
>



### 后台首页布局展示

在views/Home.vue 中设置如下内容：

```vue
<template>
  <div>
    <div id="head"></div>
    <div id="zhanghu"></div>
  </div>
</template>

<script>
export default {
  name: 'home'
}
</script>

<style lang="less" scoped>
#head {
  width: 100%;
  height: 100px;
  background-color: lightgreen;
}
#zhanghu {
  width: 100%;
  height: 260px;
  background-color: lightblue;
}
</style>

```

`效果`：

![1581150495543](img(online)/1581150495543.png)

### localStorage介入

`目标`：

​	用户登录系统后，本身的账号信息通过localStorage存储，以便业务组件访问



在views/Login.vue的登录方法中设置如下代码：

```js
login () {
  if (this.loginForm.username && this.loginForm.userpass) {
    // 通过localStorage存储登录用户的信息
    const obj = { name: 'tom', photo: 'cat.jpg', token: '!!!@@@###' }
    // localStorage只能存储字符串，引入要使用JSON.stringify()做处理
    localStorage.setItem('userinfo', JSON.stringify(obj))

    // 路由编程导航到后台首页
    this.$router.push({ name: 'home' })
  }
}
```

> 账户信息只有 name、photo、token 3样信息，这是模拟数据
>
> localStorage只能存储字符串，故要使用 JSON.stringify  转换



### 展示用户信息

目标：

​	后台首页Home.vue头部展示用户信息



在views/Home.vue 中设置如下代码

```vue
<template>
  <div>
    <div id="head">
      <p v-if="info.name">
        用户名：{{info.name}}，头像：{{info.photo}}，token：{{info.token}}
      </p>
      <p v-else>
        用户未登录系统
      </p>
    </div>
    <div id="zhanghu"></div>
  </div>
</template>

<script>
export default {
  name: 'home',
  computed: {
    info: function () {
      // 我们系统允许“非登录”用户访问，因此getItem('userinfo')有可能不会获得信息
      return JSON.parse(localStorage.getItem('userinfo') || '{}')
    }
  }
}
</script>

<style lang="less" scoped>
#head {
  width: 100%;
  height: 100px;
  background-color: lightgreen;
}
#zhanghu {
  width: 100%;
  height: 260px;
  background-color: lightblue;
}
</style>

```

> computed计算属性用于获得用户信息

效果：

![1583045814573](img(online)/1583045814573.png)



### 账户组件应用

目标：

​	创建账户组件，可以实现 查看、修改 账户信息的两个功能



步骤：

1. 创建子组件 views/Account.vue，设置如下内容

   ```vue
   <template>
     <div id="account">
       <p v-if="info.name">
         用户名：{{info.name}}，头像：{{info.photo}}，token：{{info.token}}
         <br />
         <button @click="up()">更新账户</button>
       </p>
       <p v-else>
         用户未登录系统
       </p>
     </div>
   </template>
   
   <script>
   export default {
     name: 'account',
     computed: {
       info: function () {
         // 我们系统允许“非登录”用户访问，因此getItem('userinfo')有可能不会获得信息
         return JSON.parse(localStorage.getItem('userinfo') || '{}')
       }
     },
     methods: {
       // 更新账户信息
       up () {
         const obj = { name: 'jack', photo: 'dog.jpg', token: '$$$%%%^^^' }
         localStorage.setItem('userinfo', JSON.stringify(obj))
       }
     }
   }
   </script>
   
   <style lang="less" scoped></style>
   
   ```
   
2. views/Home.vue  引入、注册、使用  Account.vue 组件

   ```vue
       <div id="zhanghu">
         <!-- 使用 -->
         <account></account>
       </div>
     </div>
   </template>
   
   <script>
   // 导入
   import Account from './Account.vue'
   export default {
     name: 'home',
     // 注册
     components: { Account },
   ```



现在后台首页展示效果：

![1583047923691](img(online)/1583047923691.png)

尴尬情况来了，单击 “更新账户” 按钮后，需要手动f5刷新，才会更新显示



## vuex介入

现在的问题是**localStorage**是浏览器技术，不是Vue的，其本身并没有“**响应式**”技术，故一个组件修改了用户信息后，另外一个组件并不能马上get(获得)到 (除非页面刷新)，现在就要使用**vuex**技术，实现多个组件之间共享数据、更新数据，还要有**响应式**效果

vuex介入后的模式是：

- 用户组件操控vuex
- vuex操控localStorage

![1581149463667](img(online)/1581149463667.png)



`目标`：

​	业务组件通过vuex操作localStorage

​	(用户不要直接面对 localStorage)

​	

`步骤`：

1. 在 store/index.js中 创建state和mutations

   ```js
   state: {
     // 用户的共享数据，名称就是user，用于页面呈现使用
     user: JSON.parse(localStorage.getItem('userinfo') || '{}')
   },
     mutations: {
       // 用户数据更新(无->有->变)
       // data: {name:xx,photo:xx,token:xx}
       updateUser: function (state, data) {
         // 1. 更新vuex自身的共享数据
         //    用于"响应式"的,   因为页面使用的都是vuex数据，即user
         //    内存数据维护，与“响应式”息息相关
         state.user = data
         // 2. 更新localStorage数据，持久更新，以便页面f5刷新后仍然可以访问到
         localStorage.setItem('userinfo', JSON.stringify(data))
       }
     },
   ```

2. Login.vue 通过vuex 存储用户信息

   ```js
   login () {
     // 用户名、密码 非空即可
     if (this.loginForm.username && this.loginForm.userpass) {
       // localStorage存储账户信息
       // token:秘钥信息，服务器端给返回的
       const obj = { name: 'tom', photo: 'cat.jpg', token: '!!!@@@###' }
       // localStorage只能存储字符串，因此JSON.stringify需要转换
       // localStorage.setItem('userinfo', JSON.stringify(obj))
       this.$store.commit('updateUser', obj)
   
       this.$router.push({ name: 'home' })
       // this.$router.push('/')
     }
   }
   ```

   

3. Home.vue 通过vuex获取用户信息

   ```js
     computed: {
       info: function () {
         // 我们系统允许“非登录”用户访问，因此getItem('userinfo')有可能不会获得信息
         // return JSON.parse(localStorage.getItem('userinfo') || '{}')
         return this.$store.state.user
       }
     }
   ```
   
   
   
4. Account.vue 通过vuex 获取、修改 用户信息

   ```js
   computed: {
     info: function () {
       // 我们系统允许“非登录”用户访问，因此getItem('userinfo')有可能不会获得信息
       // return JSON.parse(localStorage.getItem('userinfo') || '{}')
       return this.$store.state.user
     }
   },
     methods: {
       // 更新账户信息
       up () {
         const obj = { name: 'jack', photo: 'dog.jpg', token: '$$$%%%^^^' }
         // localStorage.setItem('userinfo', JSON.stringify(obj))
         this.$store.commit('updateUser', obj)
       }
     }
   ```
   
   

`效果`：

![1583049965655](img(online)/1583049965655.png)

现在，一个组件对用户信息进行修改，其他组件**不用**f5刷新，就同步感知到了



超强同学总结：



作业：

在vuex案例基础上新增一个业务组件，业务名称可以是个人中心(Person.vue)，与Account并列，本身要在Home.vue里边显示使用

开发两个功能：

1. 内部通过vuex判断用户是否存在，存在-->显示“退出”按钮，不存在-->显示 “登录” 按钮

2. 实现单击“登录”按钮，跳转到登录页面

3. 实现"退出"操作，通过vuex清除账号信息，可以不用退出系统，

   但是此时各个组件就要显示“用户没有登录系统”


