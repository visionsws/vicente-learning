## vue常用方法

### 一、弹出框

field-filter是自己建的一个fieldFilter.vue页面，fieldFilterVisible为true时显示页面

```
 <a-modal
            width="900px"
            title="过滤数据"
            style="top: 100px;"
            v-model="fieldFilterVisible"
            okText="确认"
            cancelText="取消"
            @ok="handleOk"
    >
      <field-filter  ref="fieldFilterRef" :field-name-data="fieldNameData" :login-info-data="loginInfoData"  @handleFilterBack="handleFilterBack"></field-filter>
    </a-modal>
```

### 二、父页面传值子页面

1、父页面中定义数据

```
fieldNameData: [
    {key: 'name', label: '姓名'},
    {key: 'age', label: '年龄'},
    {key: 'code', label: '编号'},
    {key: 'sex', label: '性别'},
    {key: 'phone', label: '手机号'},
    {key: 'email', label: '邮箱'}],
loginInfoData: [
    {key: 'name', label: '姓名'},
    {key: 'age', label: '年龄'},
    {key: 'code', label: '编号'},
    {key: 'sex', label: '性别'},
    {key: 'phone', label: '手机号'},
    {key: 'email', label: '邮箱'}],
```

2、在弹出框的field-filter标签中传值

3、在子页面中接收传过来的值

```
 props: {
      fieldNameData: Array,
      loginInfoData: Array,
    },
```

### 三、子页面传值父页面

1、在子页面中定义方法

```
 handleFilterData(array){
     let dataResult = []
     this.$emit('handleFilterBack',dataResult)
},
```

2、在弹出框的field-filter标签中接收方法`handleFilterBack`

```
 <field-filter  ref="fieldFilterRef" :field-name-data="fieldNameData" :login-info-data="loginInfoData"  @handleFilterBack="handleFilterBack"></field-filter>
```

3、调用父页面的方法处理传过来的数据

```
handleFilterBack(target){
      console.log(target);
      this.fieldFilterVisible = false
    },
```

### 四、父页面调用子页面中的方法

1、弹出框点击确定按钮时，调用子页面的handleFilterData方法，ref在子页面的标签中定义

```
 handleFilterOk(e) {
      this.$refs.fieldFilterRef.handleFilterData(this.dataDemo);
      this.fieldFilterVisible = false;
    },
```

this.dataDemo是传递给子页面方法的参数

### 五、数组过滤方法

1、filter方法是对数据中的元素进行过滤，也就是说是不能修改原数组中的数据，只能读取原数组中的数据，callback需要返回布尔值
为true的时候，对应的元素留下来，
为false的时候，对应的元素过滤掉

基本用法：

```
let arrObjFilter = arrObj.filter(ele => ele.age > 18)
```

进阶用法：函数接收三个参数：ele（当前遍历的数组项）index（当前项索引）arr （调用filter数组本身）

```
let arrFilter = arr.filter((ele, index, arr) => {
  return ele && ele.trim()
})
```

高级用法:
结合map使用可以先过滤出符合条件的对象然后去除某些不需要的字段,比如:

map方法对元素中的元素进行加工处理，产生一个新的数组对象。

```
let arrObjFilter = arrObj.filter(ele => ele.age > 18).map(ele => {
  return ele.name
})
```

2、13种过滤方法

```java
 // 过滤包含
      filterContain(array, key, value) {
        return array.filter(item => String(item[key]).toLowerCase().includes(String(value).trim().toLowerCase()))
      },

      // 过滤不包含
      filterNcontain(array, key, value) {
        return array.filter(item => !String(item[key]).toLowerCase().includes(String(value).trim().toLowerCase()))
      },

      // 过滤为空
      filterEmpty(array, key) {
        return array.filter(item => String(item[key]).toLowerCase().length == 0)
      },

      // 过滤不为空
      filterNempty(array, key) {
        return array.filter(item => String(item[key]).toLowerCase().length > 0)
      },

      // 过滤等于
      filterEq(array, key, value) {
        return array.filter(item => item[key] == value)
      },

      // 过滤不等于
      filterNeq(array, key, value) {
        return array.filter(item => item[key] != value)
      },

      // 过滤大于
      filterGt(array, key, value) {
        return array.filter(item => item[key] > value)
      },

      // 过滤大于等于
      filterGte(array, key, value) {
        return array.filter(item => item[key] >= value)
      },

      // 过滤小于
      filterLt(array, key, value) {
        return array.filter(item => item[key] < value)
      },

      // 过滤小于等于
      filterLte(array, key, value) {
        return array.filter(item => item[key] <= value)
      },

      // 过滤匹配
      filterMatch(array, key, value) {
        let regexp = new RegExp(value,'i');
        return array.filter(item => {
          return regexp.test(item[key])
        })
      },

      // 过滤不匹配
      filterNmatch(array, key, value) {
        let reg= new RegExp(value,'i');
        return array.filter(item => !reg.test(item[key]))
      },

      // 过滤左匹配 通过indexOf函数判断，是不是起始位置是0
      filterLmatch(array, key, value) {
        return array.filter(item => item[key].indexOf(value) == 0)
      },
```

### 六、一些其他的js方法

1、Object.keys()：es6提供的方法，用来获取对象键值对的键的集合

```
var condition={name:'a',age:13}
const filterKeys = Object.keys(condition) //['name','age']
```

2、every方法：

every()：数组的every方法，因为检查数组内的所有元素是否都满足 某一条件，如果都满足返回true,.如果有一项元素不满足就返回false。

例如：有个过滤条件是name=a and age=13，就可用every

```
 dataResult = filterKeys.every(key => {
    condition[key] == 13 //'a',13;最后dataResult是false
  })
```

3、some方法

some方法会依次执行数组的每个元素；

- 如果有一个元素满足条件，则表达式返回*true* , 剩余的元素不会再执行检测
- 如果没有满足条件的元素，则返回false

**注意：some不会改变原数组，some不会检查空数组**

```
 dataResult = filterKeys.every(key => {
    condition[key] == 13 //'a',13;最后dataResult是true
  })
```

### 七、扩展运算符 神奇的三个点点

1、数组去重

> 之前的想法可能要遍历数组去重，但是现在又es6的骚操作
```
    var arr = [1,1,2,3]
    [...new Set(arr)]  // 利用js扩展运算符的骚操作    
```
2、将一个数组放入另一个数组（打散数组）
```
    var a = [1,2]
    var b = [a,3,4]
    console.log(b) // [[1,2],3,4]
    
    // 如果使用扩展运算符
    var b = [...a, 3,4]
    console.log(b) // [1,2,3,4]
```
3、复制数组（不会改变原数组）
```
    var a = ['1','2','3']
    var b = [...a]
    console.log(b) // ['1','2','3']  
    a数组中的元素扩展为单独元素被分配到b中，可以随意改变b数组，且不会对a产生影响。
    
```
4、 拼接数组(替换concat)
```
    var a = [1,2,3]
    var b = [4,5,6]
    c  = [...a, ...b]
    a.push(...b);
    console.log(c) // [1,2,3,4,5,6]
```
5、 Math
```
    var a = [1,2,3,4,5]
    var max = Math.max(1,2,...a)
    console.log(max) // 5    
```

 6、字符串转换为数组
```
    var a = 'helloworld'
    var b = [...a]
    console.log(b) // ['h','e','l','l','o','w','o','r','l','d']
```

总结：相当于打散一个数组，显示数组中的每一个元素

### 八、方法调用

1、绑定监听@click:（以监听click为例，其他如keyup，用法类似）

```javascript
  <button @click="test1">test1</button>
  <button @click="test2('abc')">test2</button>
  <button @click="test3('abcd', $event)">test3</button>
  <checkbox @onChange="e=>test3('abcd', e)">test3</checkbox>
  methods: {
      test1(eve) {//test1函数没有参数，默认传递 $event 
        alert(eve.target.innerHTML)  //test1
      },
      test2 (msg) { //test1函数有参数，传递该参数
        alert(msg)  // abc
      },
      test3 (msg, event) { //有参数，如果想获取到enevt，则函数中需要写 $event
        alert(msg+'---'+event.target.textContent)  // abcd---test3
      }
   }
```

2、@click.stop与@click.prevent

@click.stop 阻止事件冒泡

@click.prevent 阻止事件的默认行为，

```vue
<a href="http://www.baidu.com" @click.prevent="test4">百度一下</a>   //阻止a标签跳转，仅执行函数test4

<form  action="/xxx"   @submit.prevent="test5">   //阻止表单提交，仅执行函数test5

​         <input type="submit" value="注册">
</form>
```

3、按键修饰符

@keyup.enter

//按下enter时，执行方法test7

```html
<input type="text" @keyup.enter="test7">
```

### 九、数组的一些常用方法

获取数组中的一条数据

```js
//修改选择的数据
    handleSelectChange(value, key, dataIndex) {
      const dataSource = [...this.dataSource]
      const target = dataSource.find(item => item.key === key)
      if (target) {
        target[dataIndex] = value
        this.dataSource = dataSource
      }
    },
```

删除数组中的一条数据

```js
 //删除一条数据
    onDelete(key) {
      const dataSource = [...this.dataSource]
      this.dataSource = dataSource.filter(item => item.key !== key)
    },
```

computed查询或者对某个数组的过滤

```js
computed: {
	filteredTableList(){
      let tableArray = this.dataTableList;
      if(this.isSelTable && this.isSelView){
        return tableArray;
      }
      if (!this.isSelTable && this.isSelView) {
        tableArray = this.dataTableList.filter(item => item.TABLE_TYPE !== 'BASE TABLE');
      }
      if (this.isSelTable &&  !this.isSelView) {
        tableArray = this.dataTableList.filter(item => item.TABLE_TYPE !== 'VIEW');
      }
      if (!this.isSelTable &&  !this.isSelView) {
        tableArray = [];
      }
      // 返回过来后的数组
      return tableArray
    },
}
```

watch一个数组

```js
watch: {
    selectAttrConfig: {
      handler: function(newvalue, oldvalue) {},
      //数组或者对象需要深度监控，基本类型就不需要
      deep: true 
    },
  }
```

### 十、正则表达式

获取到sql中的自定义参数age和balance

```js
let sql = "select * from test_data where age='${age}' and balance = '${balance}'"
let myreg = /(?<=\$\{).*?(?=\})/gi;
let res = sql.match(myreg);
for(let i in res){
   let itemName = res[i]
}
```

**字符串的正则方法有：**match()、replace()、search()、split()

正则对象的方法有：exec（）、test（）

**1.match**
match方法属于String正则表达方法. 
语法: str.match(regexp)
str:要进行匹配的字符串. regexp:一个正则表达式(或者由RegExp()构造成的正则表达式)
match的用法主要区分就是,正则表达式是否有全局标示g.
1)如果有g全局标志,那么返回的数组保存的是,所有匹配的内容，不包过子匹配。

2)如果没有g全局标志,那么返回的数组arr.arr[0]保存的是完整的匹配.arr[1]保存的是第一个括号里捕获的字串,依此类推arr[n]保存的是第n个括号捕获的内容.也就是当包含有全局的标志时则返回的结果第一个是正确匹配的结果，后面依次是子匹配的结果。

**2.exec**

与match方法不同exec属于正则表达式的方法.
语法:var result1 = *regexp*.exec(*str*);
regexp:正则表达式（可以直接定义也可以利用RegExp的方式定义） str:要匹配的字串
exec与match的关联就是exec(g有没有都无影响)就等价于不含有g全局标志的match.即返回数组arr[0]为匹配的完整串.其余的为括号里捕获的字符串（当含有子匹配时）.

 

1、如果exec执行的正则表达式没有子表达式(小括号内的内容，如/abc(\s*)/中的(\s*) )，如果有匹配，就返回第一个匹配的字符串内容，此时的类数组中的第一个元素为匹配的内容，（类数组中还包含有index：匹配字符串在原始字符串中的位置，input：输入的字符串）如果没有匹配返回null;

```
var reg = new RegExp("abc") ;
var str = "3abc4,5abc6";
alert(reg.exec(str));
alert(str.match(reg));
```

​    执行如上代码，你会发现两者内容均为一样：abc，此时exec 中没有子表达式同时两者均为非全局的匹配

```
var rel=/([a-z]+)\s([a-z]+)/;
var text="alen turing";
var arr=rel.exec(text);//exec返回一个数组，包含匹配到字符串以及分组(子串)里的值
console.log(arr);	//["alen turing","alen","turing"]
console.log(arr[0]);	//"alen turing"
console.log(arr[1]);	//"alen"
console.log(arr[2]);	//"turing"
console.log(RegExp.$1);	//"alen"
console.log(RegExp.$2);	//"turing"
```

**贪婪匹配与惰性匹配**

**贪婪匹配：尽可能多的匹配**

```
var regex = /\d{2,5}/g;
var test = "123 1234 12345 123456";
console.log(test.match(regex)); // ["123", "1234", "12345"]

其中正则 /\d{2,5}/，表示数字连续出现 2 到 5 次。会匹配 2 位、3 位、4 位、5 位连续数字。
```

**惰性匹配：尽可能少的匹配**

```
var regex = /\d{2,5}?/g;
var test = "123 1234 12345 123456";
console.log(test.match(regex)); // ["12", "12", "34", "12", "34", "12", "34","56"]

其中 /\d{2,5}?/ 表示，虽然 2 到 5 次都行，当 2 个就够的时候，就不再往下尝试了。
```

**通过在量词后面加个问号就能实现惰性匹配，因此所有惰性匹配情形如下:**

| 惰性量词 | 贪婪量词 |
| -------- | -------- |
| {m,n}?   | {m,n}    |
| {m,}?    | {m,}     |
| ??       | ?        |
| +?       | +        |
| *?       | *        |

**TIP** 对惰性匹配的记忆方式是:量词后面加个问号，问一问你知足了吗，你很贪婪吗?

### 十一、拖拽功能

报表分组时，将报表拖拽到哪个组中，就是分组到那个组。

一、可拖拽

那么我们对需要拖拽的div进行授权，设置draggable="true"允许其被拖动

二、定义拖拽事件

由于要对小方块进行拷贝，因此我们可以直接在拖拽开始的事件中对小方块进行拷贝

```
<div class="project_right_body_item_img" draggable="true"  @dragstart="dragReport(item.reportId,$event)">
```

dragstart就是开始拖拽前需要做的事情，在这里是将一个参数放到事件中，使事件后续能获取到这个数据

```
 dragReport(reportId,event)
      {
        console.log("我拖动了");
        event.dataTransfer.setData("Text",reportId);
      },
```

三、容器的操作

对于容器而言，我们需要对其授权，操作dragover拖拽结束的事情，允许他被放下拖动的小方块。

```
 <div @click="getReportByGroupId(item.groupId)"  @drop="drop(item.groupId,$event)" @dragover="allowDrop"></div>
```

这里定义的allowDrop就是允许拖拽的div放到这个div里面

```
  //容器允许放下
  allowDrop(event)
  {
  event.preventDefault();
  console.log("allowDrop")
  },
```

备注：此事件是通过阻止原生事件来允许容器被放下拖拽的小方块。

四、拷贝事件

放置事件drop就是拖拽过去发生的事情了，我们需要在容器的drop事件中定义好拖拽结束之后发生的事件，也就是我们需要在报表拖拽到分组里面，修改分组信息

```js
  drop(groupId,event){
  		//let treeNode = ev.target
        event.preventDefault();
        let  reportId = event.dataTransfer.getData("Text");
        console.log("要更改的大屏分组")
        console.log(reportId);
        this.updateReportGroup(reportId,groupId);
      },
```

备注：在drop事件中，首先要阻止原生父事件

总结：拖拽只需要定义好拖拽者允许拖拽draggable，容器允许被放置，同时定于好开始拖拽dragstart的事件以及拖拽结束dragover的事件，最后定义好放置事件drop即可完成。

### 十二、页面的跳转

在vue项目中通过this.$router.push路由跳转页面传递参数的方式经常用到，一般有两种方式：

1.name+params传参方式：[name为要跳转的路由名，params为要传递的参数]

```
this.$router.push({name:'success',params:{username:'tom',value:'04652'}});
```

注意：如果要传递多个参数，可以先封装成对象传递
在success页面接收参数的方法：

```
create(){
    this.username = this.$route.params.username 
}
```

2.path+query传参方式：[path为要跳转的路由路径，query为要传递的参数]

```
this.$router.push({path:'/login/success',query:{username:'tom',value:'04652'}});
```

接收：this.$route.query.username 

 ps：这两种传参方式最主要的区别就是query传参参数拼接到浏览器url地址中，而params不会。

根据实际应用场景来使用这两种方式

### 十三、store全局变量

简单的理解就是你在state中定义了一个数据之后，你可以在所在项目中的任何一个组件里进行获取、进行修改，并且你的修改可以得到全局的响应变更。下面咱们一步一步地剖析下vuex的使用:
首先要安装、使用 vuex

```
npm install vuex --save
```

然后 在src文件目录下新建一个名为store的文件夹，为方便引入并在store文件夹里新建一个index.js,里面的内容如下:

```
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
export default new Vuex.Store({
  state: {
    //这里放全局参数
  },
  mutations: {
    //这里是set方法
  },
 getters: {        
 	//这里是get方法   
 },
  actions: {},
  modules: {}
}）
```

接下来，在 main.js里面引入store，然后再全局注入一下，这样一来就可以在任何一个组件里面使用this.$store了：

```
import store from './store'//引入store
 
new Vue({
  el: '#app',
  router,
  store,//使用store
  template: '<App/>',
  components: { App }
})
```

说了上面的前奏之后，接下来就是纳入正题了，就是开篇说的state的玩法。回到store文件的index.js里面，我们先声明一个state变量，并赋值一个空对象给它，里面随便定义两个初始属性值；然后再在实例化的Vuex.Store里面传入一个空对象，并把刚声明的变量state仍里面：

```js
export default new Vuex.Store({
  state: {
    //这里放全局参数
    //要设置的全局访问的state对象
     showFooter: true,
     changableNum:0
  },
  mutations: {
     show(state) {   //自定义改变state初始值的方法，这里面的参数除了state之外还可以再传额外的参数(变量或对象);
        state.showFooter = true;
    },
    hide(state) {  //同上
        state.showFooter = false;
    },
    newNum(state,sum){ //同上，这里面的参数除了state之外还传了需要增加的值sum
       state.changableNum+=sum;
    }
  },
 getters: {        
 	//实时监听state值的变化(最新状态)
    isShow(state) {  //方法名随意,主要是来承载变化的showFooter的值
       return state.showFooter
    },
    getChangedNum(){  //方法名随意,主要是用来承载变化的changableNum的值
       return state.changebleNum
    } 
 },
    actions: {
        hideFooter(context) {  //自定义触发mutations里函数的方法，context与store 实例具有相同方法和属性
        context.commit('hide');
    },
    showFooter(context) {  //同上注释
        context.commit('show');
    },
    getNewNum(context,num){   //同上注释，num为要变化的形参
        context.commit('newNum',num)
     }
    },
  modules: {}
}）
```

**mutattions**

mutattions也是一个对象，这个对象里面可以放改变state的初始值的方法，具体的用法就是给里面的方法传入参数state或额外的参数,然后利用vue的双向数据驱动进行值的改变，同样的定义好之后也把这个mutations扔进Vuex.Store里面

这时候你完全可以用 this.$store.commit('show') 或 this.$store.commit('hide') 以及 this.$store.commit('newNum',6) 在别的组件里面进行改变showfooter和changebleNum的值了，

但这不是理想的改变值的方式；因为在 Vuex 中，mutations里面的方法 都是同步事务，意思就是说：比如这里的一个this.$store.commit('newNum',sum)方法,两个组件里用执行得到的值，每次都是一样的，这样肯定不是理想的需求

**Action **

好在vuex官方API还提供了一个actions，这个actions也是个对象变量，最大的作用就是里面的Action方法 可以包含任意异步操作，这里面的方法是用来异步触发mutations里面的方法，actions里面自定义的函数接收一个context参数和要变化的形参，context与store实例具有相同的方法和属性，所以它可以执行context.commit(' '),然后也不要忘了把它也扔进Vuex.Store里面：

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

而在外部组件里进行全局执行actions里面方法的时候，你只需要用执行

this.$store.dispatch('hideFooter')

或this.$store.dispatch('showFooter')

以及this.$store.dispatch('getNewNum'，6) //6要变化的实参

这样就可以全局改变改变showfooter或changebleNum的值了

**getters**

在组件中需要引入store定义的数据的方法：

```js
import { mapGetters } from 'vuex'
computed: {
    ...mapGetters([
    // 把 `this.showFooter` 映射为 `this.$store.getters.showFooter`
      'showFooter',
      'changableNum'
    ])
  },
```



## CSS方面的方法

### 一、一个长框带数据

```html
<div style="margin-top:10px; height: 500px">
    <!--左侧页面 -->
    <div style="float:left;border:1px solid #CDCDCD;width: 30%;height: 460px;">
        <div  v-for="item in connDataList" :key="item.id" style="padding: 3px 3px 3px 5px">
            <label style="cursor:pointer;color:#fa8c16">{{item.name}}</label>
        </div>
    </div>
<div>
```

### 二、设置让鼠标放上时，变为一只“手”

````html
<label style="cursor:pointer;">{{item.name}}</label>
````



## 调用后端方法

### 一、工具类方法

编写一个相当于工具类的调用总方法commonApi.js

```js
import axios from 'axios' // 引用axios
import Qs from 'qs' //引入数据格式化

let env_config = 'test'
// axios 配置
axios.defaults.timeout = 50000 //设置接口响应时间
if ('development' == env_config) {
  //开发环境
  axios.defaults.baseURL = 'http://localhost:9302/'
} else if ('test' == env_config) {
  //测试环境
  axios.defaults.baseURL = 'http://dompapi.bluemoon.com.cn/ubu-report-admin-service/'
} else if ('production' == env_config) {
  //生产环境
  axios.defaults.baseURL = 'http://domp.bluemoon.com.cn/ubu-report-admin-service/'
} else;

// http request 拦截器，通过这个，我们就可以把Cookie传到后台
axios.interceptors.request.use(
  config => {
    /***** 供特殊请求使用  ---begin *****/
    // if (config.url === '/BM/cross') {
    //   config.headers = {
    //     'Content-Type': 'application/x-www-form-urlencoded' // 设置跨域头部
    //   }
    //   config.data = Qs.stringify(config.data)
    // } else if (config.url === '/BM/file') {
    //   // 此处设置文件上传，配置单独请求头
    //   config.headers = {
    //     'Content-Type': 'multipart/form-data'
    //   }
    // } else {
    //   //头部带数据的情况
    //   config.headers = {
    //     'Content-Type': 'application/json;charset=UTF-8'
    //   }
    //   config.data = Qs.stringify(config.data)
    // }
    /***** 供特殊请求使用 ---end *****/

    config.headers = {
      'Content-Type': 'application/json;charset=UTF-8'
    }
    return config
  },
  err => {
    return Promise.reject(err)
  }
)

// http response 拦截器
axios.interceptors.response.use(
  response => {
    // console.log('请求拦截返回参数', response)
    if (response.status === 200) {
      // 成功
      let returnCode = response.data.code
      console.log('成功', response)
    }
    return response
  },
  error => {
    // console.log(error.toString().trim())
    if (error.toString().trim() === 'Error: Network Error') {
      console.log('网络请求异常，请稍后重试', '出错了')
    }
    return Promise.reject(error.response.data)
  }
)

export default axios

/**
 * fetch 请求方法
 * @param url
 * @param params
 * @returns {Promise}
 */
export function get(url, params = {}) {
  return new Promise((resolve, reject) => {
    axios
      .get(url, {
        params: params
      })
      .then(response => {
        resolve(response.data)
      })
      .catch(err => {
        reject(err)
      })
  })
}

/**
 * post 请求方法
 * @param url
 * @param data
 * @returns {Promise}
 */
export function post(url, data = {}) {
  console.log(data)
  return new Promise((resolve, reject) => {
    axios.post(url, data).then(
      response => {
        // console.log(response.data.code)
        resolve(response.data)
      },
      err => {
        reject(err)
      }
    )
  })
}

/**
 * patch 方法封装
 * @param url
 * @param data
 * @returns {Promise}
 */
export function patch(url, data = {}) {
  return new Promise((resolve, reject) => {
    axios.patch(url, data).then(
      response => {
        resolve(response.data)
      },
      err => {
        reject(err)
      }
    )
  })
}

/**
 * put 方法封装
 * @param url
 * @param data
 * @returns {Promise}
 */
export function put(url, data = {}) {
  return new Promise((resolve, reject) => {
    axios.put(url, data).then(
      response => {
        resolve(response.data)
      },
      err => {
        reject(err)
      }
    )
  })
}

```

### 二、具体的调用实现

