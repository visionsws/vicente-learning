## 组件使用的细节点

#### 全局组件

```
Vue.component('row',{
	data: function(){
        return {
            content:'this is content'
        }
	}
    template: '<tr><td>this is a row </td></tr>'
})
```

> <tbody>后面只能跟<tr>,所以不能直接写<row>标签，写成<tr is="row">

#### 通过ref获取dom

通过ref来获取页面上的dom

```
<div ref='hello'>hello world</div>
this.$refs.hello.innerHTML //整个Vue实例里面所有的引用里面有一个叫hello的引用，获取到对应的dom节点，也就是那个<div>
```

若在一个组件<item>中添加上ref，那么获取到的dom就是对应的组件，也就是可以获取到组件的引用。

可以获取到组件中定义的data数据

## 父子组件传值

#### 局部组件

```
var counter={
    template: '<div>0</div>'
}
var vm = new Vue({
    el: '#root',
    components: {
        counter: counter
    }
})
```

#### 父组件传值给子组件

父组件传值给子组件通过属性来传。

1. 在组件中定义一个属性

2. 子组件中通过props来接收父组件传过来的值

   ```
   <div>
   	<counter :count="1"></counter>
   </div>
   var counter={
   	props: ['count']
       template: '<div>{{count}}</div>'
   }
   ```

3. 单向数据流，子组件不能直接修改父组件的值。而是应该在自己的data上定义一个值，父组件传 

过来的值赋值给定义的值，然后修改那个值



#### 子组件向父组件传值

子组件通过事件触发来向父组件传值

```
//子组件
handleClick: function(){
    this.$emit('change',2) //子组件向外触发一个change事件，2是传过来的参数
}

//父组件
<counter :count="1" @change="handleInc"></counter>
父组件通过@change来接收到子组件传过来的信息
handleInc: function(step){  //step是子组件传过来的参数
    this.total = this.total+step
}
```



## 组件参数校验与非props特性

#### 组件参数校验

定义一个全局组件

```
Vue.component('child',{
    template: '<div>Child</div>'
})
```

父组件给子组件通过属性传递一些值，而子组件做的一些约束就是组件的参数校验

```
<child :content ="hello"></child>

Vue.component('child',{
	props: {
        content：Number //约束content传过来的数据是数字类型
        key:[ Numver, String],
        content: {
            type: String,  //content传入的类型是字符型
            required: true,  //user必须传
            default: 'default' //给user一个默认值
            validator: function(value){  //校验传进来的内容，长度大于5
                return (value.length>5)
            }
        }
	}
    template: '<div>Child</div>'
})
```

#### 非props特性

父组件要传一个名叫content的属性，而子组件刚好定义了content的prop，这个就是props特性

非props特性

1. 父组件传，但子组件不接收
2. 子组件不能使用父组件的值
3. 子组件的dom标签会显示父组件的属性

## 给组件绑定原生事件

例如：给子组件的标签中定义一个click事件

```
//下面的实现是错误的
<child @click="handleClick"></child>
methods: {
    handleClick:function(){
        alert('click')
    }
}
```

上面的实现是错误的，原因就是在子组件中定义的@click中的click是监听的自定义事件的名称，它是接收子组件触发的事件名称，如：this.$emit('click')，@click是接收这样的事件的，并不是我们熟知的点击事件

实现上面的例子

方法一：

```
Vue.component('child',{
	template: '<div @click="handleChildClick">Child</div>'  
	//这里的click就是原生的点击事件
}
```

方法二

在click中添加.native就可标明为原生事件

```
<child @click.native="handleClick"></child>
```

## 非父子组件间的传值

方法一，使用Vuex



方法二，使用发布订阅模式，也称为总线机制

1. 创建子组件child
2. 在prototype上挂载一个bus属性，指向一个vue，以后每个vue实例上都有bus属性

```
Vue.prototype.bus = new Vue()
```

3. 在子组件模板中定义一个点击事件
4. 通过bus触发事件发送信息，然后同样通过bus来接收发送的信息

```
methods: {
    handleClick: function(){
        this.bus.$emit('change',this.content)
    }
}
mounted: function(){
    var _this = this;
    //监听change事件
    this.bus.$on('change',function(msg){
        _this.content = msg
    })
}
```

#### 在Vue中使用插槽（slot）

 怎么使父组件给子组件优雅的传递dom，例如父组件要给子组件传递<p>Dell</p>

按以往的方法应该是

```
<child content='<p>Dell</p>'></child>
//接收数据
props: ['content']
template: `<div v-html='content'></div>`  //看得出来，这样每次都会多了一个<div>标签
```

使用slot可以解决问题

```
<child>
	<p>Dell</p>
</child>

template: `<div><slot></slot></div>` //slot的内容就是子组件<child>中的dom内容
```

如果template中有多个<slot>，那怎么确定哪个slot要那些dom呢

可以通过名称来确定对应的dom，如

```
<child>
	<div class='header' slot='header'>header</div>
	<div class='footer' slot='footer'>header</div>
</child>
template: `<div><slot name='header'></slot></div>` //slot的内容就是子组件<child>中的dom内容
```

#### Vue中的作用域插槽

1. 父组件调用子组件的时候，给子组件传递作用域插槽，作用域插槽必须是<template>标签开头和结尾
2. 需要告诉子组件接收到的信息都放在什么地方，如：slot-scope="props" 就是接收到的信息都放在props中
3. 需要告诉子组件需要怎样展示数据

```
<child>
	<template slot-scope="props">
		<h1>{{props.item}}</h1>
	</template>
</child>
template: '<div><slot v-for="item of list" :item=item></slot></div>'
```

#### 动态组件与v-once指令

1. 定义2个子组件child-one和child-two
2. 定义一个click事件，通过点击事件循环的切换child-one和child-two 通过v-if可以实现

通过动态组件来怎么实现呢

```
<component :is="type"></component> 
//type的值为child-one时，这里显示child-one子组件，为child-two时显示child-two的
```

切换的时候，每次都是先销毁，然后再创建子组件，每次切换都销毁和创建

v-once就是为了避免这种情况的发生，使用v-once第一次展示的时候，会将组件放到内存中，第二次就不需要创建组件了，可以直接从内存中读取到

```
template: '<div v-once>child-one</div>'
```

**参考**
[Vue.js API文档](https://cn.vuejs.org/v2/api/#transition)
[慕课网：Vue2.5开发去哪儿网App 从零基础入门到实战项目](https://coding.imooc.com/class/chapter/203.html#Anchor)