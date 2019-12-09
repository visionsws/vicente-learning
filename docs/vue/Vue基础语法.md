## Vue基础语法

1. 挂载点：el：代表所对应的document
2. 模板：template：可以在挂载点上显示一段页面代码
3. 实例数据：通过data数据和template自动生成需要显示的内容
4. 插值表达式：把数据插入到html中{{msg}}
5. v-on:绑定一个事件。如：v-on:click 绑定一个点击事件，简写：v-on=@
6. 事件触发的方法，vue.methods下的方法中
7. 属性的绑定：v-bind:可以绑定属性到页面上，此时=后的字符就不是字符串了，而是js表达式，即data下的属性，简写为：冒号，如v-bind:title = :title
8. 双向绑定：常用input来绑定某个变量，页面上input改变时，变量也发生变化 ，使用v-model，input框中的内容改变了，那data下的属性内容也跟着改变，然后有改变绑定这个属性的页面一起改变
9. 计算属性：使用computed来计算属性的变化，如两个属性双加，如果属性没有改变的话，computed不会重新计算，而是找缓存，如果两个属性中的某一个改变了，会自动重新计算结果
10. 计算属性getter和setter：

```js
computed: {
    fullName:{
        get: function(){
            return this.firstName + " " + this.lastName
        },
        set: function(value){
            var val = value.sp

        }
    }
}
```

1. 监听器：通过watch来定义。可以监听属性，如果属性改变了，就会调用函数，如：msg:function(){}，msg必须先在data中定义
2. v-if:指令将根据表达式的值(true 或 false )来决定是否显示标签
3. v-show:作用和v-if相当，也是为false时不显示所在标签，区别是，v-if是直接在dom中清除，而v-show是通过设置style属性display为none来隐藏标签
4. v-for：循环展示data属性中的list值，如：v-for="(value, key, index) in list"
5. 模板语法：差值表达式：{{msg}}; 使用v-text="msg";v-html="msg",可以使用html标签； 凡是v-XX后面跟的值都是一个js表达式如，刚刚的msg可以写msg+‘ haha’等

#### Vue中的样式绑定

每次点击一个数据，改变数据的样式，数据改变，样式就改变

> 1. 定义一个绑定事件@click="handleDivClick"
> 2. 绑定一个class样式 :class="{activated: isActivated}" v-xxx里面的都是一个js表达式，{}表明里面是一个对象，意思是class的名称是activated，而绑不绑定取决于isActivated是true还是false
> 3. 每次点击的时候，通过方法handleDivClick修改isActivated的值

方法二

> 1. 绑定class的样式，如 :class="[activated, activateOne]"，里面的activated是data中定义的一个变量
> 2. activated变量中的值就是class样式绑定的class的名称。this.activated= this.activated==="activated" ? "" : "activated"

方法三

>1. 通过使用:style="styleObj",styleObj在data中定义的数据，然后在styleObj中直接定义css的样式""
>
>   如：styleObj:{color:"black"};
>
>   例2（一个表格的单双行不同颜色）：:style="{backgroundColor:index%2===0 ? oddColor : evenColor, borderTop: lineBorder}
>
>2. style也可以写数组，如:style="[styleObj, {fontSize: '20px'}]"



#### Vue中的条件渲染

> v-if:指令将根据表达式的值(true 或 false )来决定是否显示标签
>
> v-show:作用和v-if相当，也是为false时不显示所在标签，区别是，v-if是直接在dom中清除，而v-show是通过设置style属性display为none来隐藏标签，性能比v-if低
>
> v-if,v-else-if,v-else三个对应的标签要连着一起写，中间不能有别的标签
>
> vue会尽量复用已有的dom，复用的时候可能里面会含有value值，不会自动清空。可以添加key值来避免复用dom
>
> 可以通过这2个来定义页面显不显示对应的div标签



#### Vue中的列表渲染

使用v-for来显示列表

>1. key值需要一个唯一的值，不推荐使用index作为key值，有key值能提高性能，可服用dom
>
>2. 不能通过修改数组的下标来改变数组，这样有数据，但页面不会改变
>
>3. 数组提供的方法：push，pop，shift，unshift，splice，sort，reverse
>
>4. 想改变数组某项数据，数据发生变化，页面跟着变化，可使用vm.list.splice(1,1,{id:"333"})，从第1项开始删除1项，然后添加1项；也可以通过改变数据的引用地址来达到，如：vm.list=[111,222,333]
>
>5. template 模板占用符，可以帮我们包裹一些数据，但是不在前端显示出来
>
>6. (item,key,index) of userInfo 显示对象中的每一项数据，修改某项属性：vm.userInfo.name="dell lee",添加数据：可以通过修改数据引用来，vm.userInfo = {name:"dell"};
>
>7. set方法：可以通过set方法来给对象添加值 Vue.set(vm.userInfo,"address","beijing");可以使用实力生的方法，写成：vm.$set(vm.userInfo,"address","beijing");
>
>   set方法修改数组：vm.$set(vm.userInfo,1,66)



## Vue中的组件

1. todolist功能：v-model,@click,v-for,this.list.push(this.inputvalue)
2. 组件的创建：Vue.component('todo-item') ，template,
3. 组件的使用：在html页面上可以使用<todo-item></todo-item>
4. 局部组件：var TodoItem = {}，在Vue实例里面添加components来注册TodoItem  
5. 传值到组件中props：在<todo-item>中绑定一个值到变量中，如:content，然后子组件中使用props:['content']声明和接收变量数据，然后就可以在template中使用该变量了。
6. 组件与实例：每一个组件都是一个Vue的实列
7. 子组件传值给父组件：使用发布订阅模式来完成，使用this.$emit向外发布一个事件，可带参数，父组件刚好订阅了发布的事件，其实就是一个方法

## Vue-cli的使用

1. Vue-cli简介:可以快速构建一个标准的Vue项目，同时自带 webpack各种配置，没技术门槛

2. 使用Vue-cli创建项目：vue init webpack todolist，项目名。。。

3. 在cli脚手架上data不是一个对象了，变成了一个函数，函数的返回值是我们具体的数据

4. 全局样式和局部样式：子组件的样式只会调用子组件中定义的，父组件的样式不影响子组件，但也可以加上scope

   

## Vuex

1. vuex介绍：为Vue.js开发的状态管理模式
2. 官方说明：**Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 devtools extension，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。**
3. 组件状态集中管理
4. 组件状态改变遵循统一的规则
5. state: 全局状态，如store.state.count
6. mutations: vuex改变状态的方法集合，使用store.commit('add')调用里面定义的方法
7. 使用：在main.js中挂载使用它，就是这一句：Vue.use(Vuex)，

## 调试

1. Vue的Chrome插件
2. console.log, alert(),   debugger
3. vm实例 给vue的实例化个var名称，window对象绑定



## Vue生命周期

1. 生命周期函数就是vue实例在某一个时间点会自动执行的函数
2. beforeCreate
3. created
4. beforeMount 页面还没有渲染
5. mounted 页面已经渲染完毕
6. beforeUpdate 数据改变前
7. updated 数据改变后