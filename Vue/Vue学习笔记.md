## Vue study



Vue是一套构建用户界面的框架，只关注视图层，它不仅易于上手，还便于与[第三方库](https://so.csdn.net/so/search?from=pc_blog_highlight&q=第三方库)或既有项目整合。（[Vue](https://so.csdn.net/so/search?from=pc_blog_highlight&q=Vue)有配套的第三方类库，可以整合起来做大型项目的开发）



## 一、MVVM概念

- MVC 是后端的分层开发概念；

- MVVM是前端视图层的概念，主要关注于 视图层分离。MVVM把前端的视图层分为了 三部分 ：Model, View , VM ViewModel

  **M：Model 数据模型**。数据层，数据可能是固定的数据, 更多的是来自服务器, 从网络上请求下来的数据

  **V: View 视图模板**。视觉层，在前端开发中, 通常是DOM层，作用: 是给用户展示各种信息

  **VM： View-Model 视图模型**。视图模型层，是View和Model沟通的桥梁。一方面实现了Data Binding (数据绑定), 将Model的改变实时的反应到View中；另一方面实现了DOM Listener (DOM监听), 当DOM发生一些时间 (点击, 滚动, touch等) 时, 可以监听到, 并在需要的情况下改变对应的Data

## 二、快速开始

要使用Vue，首先需要在HTML文件中引入vue.js

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```



## 三、Vue基本代码结构

<center style="background:#eee"><img src="https://img-blog.csdnimg.cn/20200824125405384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NTQ2Nzg3,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" /></center>



```vue
const vm = new Vue({
	el:'#app',//所有的挂载元素会被 Vue 生成的 DOM 替换
	data:{ // this->window },
	methods:{ // this->vm},
	//注意，不应该使用箭头函数来定义 method 函数 ,this将不再指向vm实例
	props:{} ,// 可以是数组或对象类型，用于接收来自父组件的数据
	//对象允许配置高级选项，如类型检测、自定义验证和设置默认值
	watch:{ // this->vm},
	computed:{},
	render(){},
	// 声明周期钩子函数
});
```

当一个 Vue 实例被创建时，它将 data 对象中的所有的 property 加入到 Vue 的响应式系统中。当这些 property 的值发生改变时，视图将会产生“响应”，即匹配更新为新的值。

### 实例属性和方法

* （1）el属性：表示此vue实例需要控制的页面那个区域，值是元素id
* （2）data属性：表示控制区域中要用到的数据。注意：以 `_ `或` $` 开头的 property 不会被 Vue 实例代理，因为它们可能和 Vue 内置的 property、API 方法冲突。你可以使用例如 `vm.$data._property` 的方式访问这些 property。
* （3）访问data中定义的变量 ： `vm.a , vm.$data.a`
* （4）methods属性：定义el控制的区域，需要执行的动作。访问方法可以直接：`vm.方法名()`
* （5）watch属性：`vm.$watch()`

## 四、Vue指令

### 4.1 基本语法

#### （1）差值表达式 `{{data}}`

* Mustache语法 (双大括号)
* 可以直接写变量
* 可以写简单的表达式

#### （2） v-text

* 与Mustache相似, 一般不用, 不灵活，但是v-text没有插值表达式直接显示出未编译的Mustache标签的问题

#### （3） v-html

* 后面往往跟一个string类型，会将string的html解析出来并渲染

####  （4）v-cloak

* 在某些情况下, 我们浏览器可能会直接显示出未编译的Mustache标签

#### （5）v-pre

* 用于跳过这个元素和它子元素的编译过程, 用于显示原本的Mustache语法

#### （6）v-once

* 后面不需要跟任何表达式，表示元素和组件只渲染一次, 不会随着数据的改变而变化

#### （6）v-bind

* 作用: 动态绑定属性，简写 `::`

示例：

```vue
<div id="app">
    <h1>{{ message }}</h1>
    <h1 v-cloak>{{ message }}</h1>

    <h1 v-text="message"></h1>

    <!-- v-text 会将html文本当做普通字符串处理-->
    <p v-text="message2"></p>
    <!-- v-html 会将html文本就当做html文本处理-->
    <p v-html="message2"></p>

    <!--v-bind 的两种写法，效果是一样的-->
    <input type="button" v-bind:value="btn_value">
    <input type="button" :value="btn_value">
</div>

<script type="text/javascript">
    const vm = new Vue({
        el: '#app',
        data: {
            message: 'Hello World!',
            message2: '<button type="button" >click me</button>',
            btn_value: "点击跳转"
        }
    });
</script>
```



### 4.2 表单绑定

#### v-model表单绑定

指令可以实现表单元素和数据的双向绑定

示例：

```vue
<div id="app">

    <!--1、v-model 配合普通文本框使用-->
    <input type="text" v-model:value="username"/>
    <p>{{ username }}</p>

    <!--2、配合textarea 使用-->
    <textarea v-model="message"></textarea>
    <p>输入的内容: {{ message }} </p>

    <!--3、配合radio使用-->
    <input type="radio" id="male" value="男" v-model="sex"/>
    男
    <input type="radio" id="female" value="女" v-model="sex"/>
    女
    <h2>您选择的性别是: {{ sex }}</h2>

    <!--4、配合单选框使用-->
    <label for="agree">
        <input type="checkbox" id="agree" v-model="isAgree">同意协议
    </label>
    <input type="button" value="下一步" :disabled="!isAgree">

    <br/>

    <!--5、v-model配合下拉框-->
    <select v-model="mySelect">
        <option value="苹果">苹果
        <option>
        <option value="橘子">橘子
        <option>
        <option value="香蕉">香蕉
        <option>
    </select>
    <p>您最喜欢的水果: {{ mySelect }}</p>

</div>

<script type="text/javascript">
    const vm = new Vue({
        el: '#app',
        data: {
            username: "法外狂徒—张三",
            message: '随便写的东西吧',
            sex: '男',
            isAgree: false,
            mySelect: '苹果',
            mySelect2: '苹果'
        },
        methods: {}
    });
</script>
```



### 4.3 条件渲染

#### （1）v-if、v-else-if、v-else

* 控制元素是否显示，当条件为`true`时会将对应标签显示出来，为`false`时则消失。

#### （2）v-show

* **v-show**与`v-if`功能一样可以控制隐藏与显示，但是不同点是`v-if`每次都会重新删除或者创建元素，而`v-show`则不会每次都进行DOM的删除和创建操作只是添加了`display:none`样式。

* 当条件为false的时：

  * v-if: 指令的元素, 不会渲染到dom中
  * v-show: dom增加一个行内样式display: none

  基于此原理：**`v-if`有较高的切换性能消耗。`v-show`有较高的初始化渲染消耗。如果元素涉及到频繁的切换，最好不要使用`v-if`,如果元素可能永远也不会被显示出来被用户看到则推荐使用`v-if`**

```vue
<div id="app">

    <!--v-if 和 v-show 控制元素的显示和隐藏-->
    <input type="button" :value="toggle" @click="toggleBtn"/>
    <div v-if="!isDisable">这是一段文字</div>
    <div v-else>另一段文字</div>
    <div v-show="!isDisable">这是一段文字</div>
</div>

<script type="text/javascript">
    const vm = new Vue({
        el: '#app',
        data: {
            isDisable: false,
            toggle: "隐藏"
        },
        methods: {
            toggleBtn() {
                this.isDisable = !this.isDisable;
                this.toggle = this.isDisable ? "显示" : "隐藏"
            }
        }
    });
</script>
```



### 4.4 事件处理

#### （1）v-on

可以用 `v-on` 指令监听 DOM 事件，并在触发时运行一些 JavaScript 代码。

* 作用: 绑定事件监听，简写: `@`
  * 没有参数的情况下, 可以不写(); 如果方法本身有一个参数, 会默认将原生事件event参数传递进去
  * 如果传入某个参数, 同时需要event时, 可以通过`$event`传入事件

  

#### （2）事件修饰符

<table><thead><tr><th>修饰符</th><th>作用</th><th>实际调用</th></tr></thead><tbody><tr><td>.stop</td><td>阻止事件冒泡</td><td>event.stopPropagation()</td></tr><tr><td>.prevent</td><td>阻止默认事件</td><td>event.preventDefault()</td></tr><tr><td>{keyCode I keyAlias}</td><td>监听某个键盘的键帽</td><td>–</td></tr><tr><td>.native</td><td>监听组件根元素的原生事件</td><td>–</td></tr><tr><td>.once</td><td>只触发一次回调</td><td>–</td></tr></tbody></table>

示例：

```vue
<div id="app">

    <!--v-on 的用法示例：两种写法效果一样-->
    <input type="button" :value="btn_value" v-on:click="show"/>
    <input type="button" :value="btn_value" @click="show">
</div>

<script type="text/javascript">
    const vm = new Vue({
        el: '#app',
        data: {
            btn_value: 'click me'
        },
        methods:{
            show: function (){
                alert("点击了按钮")
            }
        }
    });
</script>
```

#### （3）按键修饰符

在监听键盘事件时，我们经常需要检查详细的按键。Vue 允许为 `v-on` 在监听键盘事件时添加按键修饰符：

```vue
<div id="app">
    <label for="username">
        <input name="username" id="username" v-model="username" @keyup.enter="add">
    </label>

    <div>
        输入的历史：
        <li v-for="item in list" :key="item.username">{{item}}</li>
    </div>
</div>

<script type="text/javascript" >
    const vm = new Vue({
        el: '#app',
        data: {
            username: '',
            list: []
        },
        methods: {
            add(username) {
                this.list.push(this.username);
            }
        }
    });
</script>
```

为了在必要的情况下支持旧浏览器，Vue 提供了绝大多数常用的按键码的别名：

- `.enter`
- `.tab`
- `.delete` (捕获“删除”和“退格”键)
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

##### 自定义按键修饰符

除过vue提供的按键修饰符，我们还可以通过全局 `config.keyCodes` 自动以按键修饰符：

```js
Vue.config.keyCodes.f1 = 112
```



### 4.5  列表渲染

#### v-for遍历数组

v-for可以用于遍历数组，对象等集合。官方推荐, 使用v-for的时候, 加上一个 key属性。key的作用是为了高效的更新虚拟DOM，key要具有唯一性, 不然就没意义

```vue
<div id="app">

    <!--1、遍历普通数组-->
    <div><span v-for="num in list" :key="num">{{ num }} </span></div>
    <div><span v-for="(num,index) in list" :key="num">{{index}}:{{ num }} </span></div>
    <!--2、遍历对象-->
    <div><span v-for="(value,key) in userinfo" :key="key">{{key}}-{{ value }} </span></div>
</div>

<script type="text/javascript">
    const vm = new Vue({
        el: '#app',
        data: {
            list: [1, 2, 3, 4, 5, 6, 7],
            userinfo: {username:"法外狂徒",job:"游走在作死边缘",age:"NA"}
        },
        methods: {}
    });
</script>
```



### 4.6 自定义指令

除了核心功能默认内置的指令 (`v-model` 和 `v-show`)，Vue 也允许注册自定义指令，比如要实现一个让input文本框实现页面加载后自动聚焦的指令 `v-focus`：

```vue
<div id="app">
    <input type="text" v-focus>
</div>

<script type="text/javascript">

    //全局指令 第一个参数是指令的名称 第二个参数是指令的回调钩子函数对象，根据需要实现vue给定不同阶段的钩子函数
    Vue.directive('focus',{
        inserted(e){
           e.focus();
        }
    })

    const vm =new Vue({
        el: '#app',
    });
</script>
```

同样，也可以使用`directives`属性在一个vue实例中定义多个局部指令：

```vue
<div id="app">
    <input type="text" v-focus>
</div>

<script type="text/javascript">

    const vm =new Vue({
        el: '#app',
        directives: {
            //局部指令
            focus: {
                inserted(e){
                    e.focus();
                }
            }
        }
    });
</script>
```

#### 钩子函数

一个指令定义对象可以提供如下几个钩子函数 (均为可选)：

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。
- `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

#### 钩子函数参数

指令钩子函数会被传入以下参数：

- `el`：指令所绑定的元素，可以用来直接操作 DOM。
- `binding`：一个对象，包含以下 property：
  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。
- `vnode`：Vue 编译生成的虚拟节点。
- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。



#### 动态指令参数

使用动态指令参数可以使得我们的自定义指令更加灵活和具有可复用性。比如下面这个例子：自定义v-print指令，可以实现根据指令参数化arg实现动态控制字体大小 

```vue
<div id="app">
    <input type="text" v-focus v-model="message">

    <select v-model="fontsize">
        <option value="10">10 <option>
        <option value="20">20 <option>
        <option value="30">30<option>
        <option value="40">40<option>
    </select>

    <div v-print:[fontsize]="message"></div>
</div>

<script type="text/javascript">

    Vue.directive('focus', {
        inserted(e) {
            e.focus();
        }
    });

    <!-- v-print 指令可以根据arg参数化参数动态调整字体大小 -->
    Vue.directive('print', {
        inserted: function (el, binding, vnode) {
            el.style.fontSize = binding.arg + "px";
            el.innerHTML = binding.value;
        },
        update: function (el, binding, vnode) {
            el.style.fontSize = binding.arg + "px";
            el.innerHTML = binding.value;
        }
    })

    const vm = new Vue({
        el: '#app',
        data: {
            fontsize: '30',
            message: "hello"
        }
    });
</script>
```



## 五、Class 与 Style 绑定



## 六、Vue过滤器

**过滤器可被用于一些常见的文本格式化**。过滤器可以用在两个地方：**双花括号插值和 `v-bind` 表达式** (后者从 2.1.0+ 开始支持)。过滤器应该被添加在 JavaScript 表达式的尾部并且由“管道”符号指示：

```vue
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```

管道符左边的值会自动作为管道符右边的过滤器的默认入参，比如 `{{ message | capitalize}}` 中过滤器 `capitalize` 的就默认由一个参数是message的值。

#### 5.1 全局过滤器

```vue
<div id="app">

    <table border="1">
        <thead>
        <tr>
            <th>id</th>
            <th>姓名</th>
            <th>出生日期</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(item) in list" :key="item.key">
            <td>{{ item.id }}</td>
            <td>{{ item.name }}</td>
            <!--时间过滤器对时间进行格式化操作-->
            <td>{{ item.birth | timeFormat }}</td>
        </tr>
        </tbody>
    </table>
</div>

<script type="text/javascript">

    //时间格式化器 全局过滤器
    Vue.filter('timeFormat', function (datestr) {
        if (datestr != null) {
            var date = new Date(datestr);
            return date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate() +
                ' ' + date.getHours() + ':' + date.getMinutes() + ":" + date.getSeconds();
        }
        return datestr;
    })

    const vm = new Vue({
        el: '#app',
        data: {
            list: [
                {id: 1, name: '张三', birth: new Date(1998, 11, 11, 8, 0, 0)},
                {id: 2, name: '李四', birth: new Date(1999, 11, 14, 8, 0, 0)},
                {id: 3, name: '王富贵', birth: new Date(2000, 10, 14, 8, 0, 0)},
                {id: 4, name: '来福', birth: new Date(1982, 10, 30, 8, 0, 0)}
            ]
        }
    });
    
</script>
```



#### 5.2 局部过滤器

```vue
<div id="app">

    <table border="1">
        <thead>
        <tr>
            <th>id</th>
            <th>姓名</th>
            <th>出生日期</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(item) in list" :key="item.key">
            <td>{{ item.id }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.birth | formatTime }}</td>
        </tr>
        </tbody>
    </table>
</div>

<script type="text/javascript">
    const vm = new Vue({
        el: '#app',
        data: {
            list: [
                {id: 1, name: '张三', birth: new Date(1998, 11, 11, 8, 0, 0)},
                {id: 2, name: '李四', birth: new Date(1999, 11, 14, 8, 0, 0)},
                {id: 3, name: '王富贵', birth: new Date(2000, 10, 14, 8, 0, 0)},
                {id: 4, name: '来福', birth: new Date(1982, 10, 30, 8, 0, 0)}
            ]
        },
        filters: {
            //时间格式化器 局部过滤器
            formatTime(datestr){
                if (datestr != null) {
                    var date = new Date(datestr);
                    return date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate() +
                        ' ' + date.getHours() + ':' + date.getMinutes() + ":" + date.getSeconds();
                }
                return datestr;
            }
        }
    });

</script>
```



特别的，**当全局过滤器和局部过滤器重名时，会采用局部过滤器。**另外，过滤器可以串联：

```js
{{ message | filterA | filterB }}
```

在这个例子中，`filterA` 被定义为接收单个参数的过滤器函数，表达式 `message` 的值将作为参数传入到函数中。然后继续调用同样被定义为接收单个参数的过滤器函数 `filterB`，将 `filterA` 的结果传递到 `filterB` 中。





## 七、Vue生命周期

<img src="https://upload-images.jianshu.io/upload_images/11370083-f279314aef6741db.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1080/format/webp" style="width:50%;" />

#### beforeCreate( 创建前 )

在实例初始化之后，数据观测和事件配置之前被调用，此时组件的选项对象还未创建，el 和 data 并未初始化，因此无法访问methods， data， computed等上的方法和数据。

#### created ( 创建后 ）

实例已经创建完成之后被调用，在这一步，实例已完成以下配置：数据观测、属性和方法的运算，watch/event事件回调，完成了data 数据的初始化，el没有。 然而，挂在阶段还没有开始, $el属性目前不可见，这是一个常用的生命周期，因为你可以调用methods中的方法，改变data中的数据，并且修改可以通过vue的响应式绑定体现在页面上，，获取computed中的计算属性等等，通常我们可以在这里对实例进行预处理，也有一些童鞋喜欢在这里发ajax请求，值得注意的是，这个周期中是没有什么方法来对实例化过程进行拦截的，因此假如有某些数据必须获取才允许进入页面的话，并不适合在这个方法发请求，建议在组件路由钩子beforeRouteEnter中完成

#### beforeMount（挂载前）

挂载开始之前被调用，相关的render函数首次被调用（虚拟DOM），实例已完成以下的配置： 编译模板，把data里面的数据和模板生成html，完成了el和data 初始化，注意此时还没有挂在html到页面上。

#### mounted（挂载后）

挂载完成时会调用，也就是模板中的HTML渲染到HTML页面中，此时一般可以做一些ajax操作，mounted只会执行一次。

#### beforeUpdate（更新前）

在数据更新之前被调用（此时内存中的数据已经被更新，只是还没被刷新到页面），可以在该钩子中进一步地更改状态，不会触发附加地重渲染过程

#### updated（更新后） 

在由于数据更改导致地虚拟DOM重新渲染时会调用，调用时组件DOM已经更新，所以可以执行依赖于DOM的操作，但是在大多是情况下，应该避免在此期间更改状态，因为这可能会导致更新无限循环，该钩子在服务器端渲染期间不被调用

#### beforeDestroy（销毁前）

在实例销毁之前调用，实例仍然完全可用，

1. 这一步还可以用this来获取实例，
2. 一般在这一步做一些重置的操作，比如清除掉组件中的定时器 和 监听的dom事件

#### destroyed（销毁后）

在实例销毁之后调用，调用后所有的事件监听器会被移出，所有的子实例也会被销毁，该钩子在服务器端渲染期间不被调用





## 八、Vue发送Ajax请求

### 8.1 vue-resource

vue-resource是Vue.js的一款插件，它可以通过XMLHttpRequest或JSONP发起请求并处理响应，也就是说，jQuery中ajax能做的事情，vue-resource插件一样也能做到，而且vue-resource的API更为简洁。此外，vue-resource还提供了非常有用的inteceptor功能，使用inteceptor可以在请求前和请求后附加一些行为，比如使用inteceptor在ajax请求时显示loading界面。

vue-resource GitHub开源地址：https://github.com/pagekit/vue-resource

#### 安装方法

* 使用CDN

```html
<script src="https://cdn.jsdelivr.net/npm/vue-resource@1.5.3"></script>
```

* 使用命令安装

```shell
#arn
yarn add vue-resource
#npm
npm install vue-resource
```



#### 基本使用

```js
// 全局引用
Vue.http.get('someurl', [options]).then(successCallback, errorCallback);
Vue.http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);

// 基于某个Vue实例使用
var vm = new Vue();
vm.$http.get('/someUrl', [options]).then(successCallback, errorCallback);
vm.$http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);
```



**示例：发送GET请求**

```js
<div id="app">

    <table border="1">
        <thead>
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>地址</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(item) in list" :key="item.name">
            <td>{{ item.name }}</td>
            <td>{{ item.age }}</td>
            <td>{{ item.address }}</td>
        </tr>
        </tbody>
    </table>
</div>

<script type="text/javascript">

    var vm = new Vue({
        el: '#app',
        data: {
            list: [
                {
                    "name": "法外狂徒1",
                    "address": "本宇宙-拉尼亚凯亚超星系团-室女座星系团-本星系群-银河系-猎户座旋臂-太阳系-地球",
                    "age": 100
                }
            ]
        },
        methods: {
            getUserInfoLists() {
                this.$http.get("http://127.0.0.1:8001/v1/demo/list")
                    .then(function (response) {
                            console.log(response);
                            if (response.data.success) {
                                this.list = response.body.data;
                            }
                        },
                        function (error) {
                            console.log(error.message);
                        });
            }
        },
        created(){
            this.getUserInfoLists();
        }
    })
</script>
```



**示例：发送POST请求**

```js
<div id="app">
    <form>
        <input type="text" name="username" v-model:value="username" placeholder="username">
        <input type="text" name="address" v-model:value="address" placeholder="address">
        <input type="text" name="age" v-model:value="age" placeholder="age">
        <input type="button" @click="submit" value="提交">
    </form>

    <p>你提交的信息：{{info}}</p>
</div>

<script type="text/javascript">

    const vm = new Vue({
        el: '#app',
        data: {
            username: '',
            address: '',
            age: '',
            info:''
        },
        methods: {
            submit() {
                let username = this.username;
                let address = this.address;
                let age = this.age;
                this.$http.post("http://127.0.0.1:8001/v1/demo/add",
                    JSON.stringify({
                        "username":username,
                        "address":address,
                        "age":age
                    })).then((response)=>{
                    //接口响应成功 2xx
                    var body = response.body;
                    if(body.success) {
                        //接口返回处理成功
                        this.info = response.body.data;
                    }else{
                        //接口返回处理失败
                        this.info= body.message;
                    }
                },(error)=>{
                    //接口响应失败 4xx 5xx
                    console.log(error);
                })
            }
        }
    })
</script>
```



除了GET和POST请求，vue-resource还提供了一下几种常见的请求方式，如下所示：

- `head(url, [options])`
- `delete(url, [options])`
- `jsonp(url, [options])`
- `put(url, [body], [options])`
- `patch(url, [body], [options])`

值得注意的是API中的options属性，发送请求时的options选项对象包含以下属性：

|    参数     |             类型              |                             描述                             |
| :---------: | :---------------------------: | :----------------------------------------------------------: |
|     url     |           `string`            |                          请求的URL                           |
|   method    |           `string`            |      请求的HTTP方法，例如：'GET', 'POST'或其他HTTP方法       |
|    body     | `Object`, `FormData` `string` |                         request body                         |
|   params    |           `Object`            |                      请求的URL参数对象                       |
|   headers   |           `Object`            |                        request header                        |
|   timeout   |           `number`            |        单位为毫秒的请求超时时间 (`0` 表示无超时时间)         |
|   before    |      `function(request)`      |      请求发送前的处理函数，类似于jQuery的beforeSend函数      |
|  progress   |       `function(event)`       | [ProgressEvent](https://developer.mozilla.org/en-US/docs/Web/API/ProgressEvent)回调处理函数 |
| credentials |           `boolean`           |                表示跨域请求时是否需要使用凭证                |
| emulateHTTP |           `boolean`           | 发送PUT, PATCH, DELETE请求时以HTTP POST的方式发送，并设置请求头的`X-HTTP-Method-Override` |
| emulateJSON |           `boolean`           | 将request body以`application/x-www-form-urlencoded` content type发送（表单形式发送） |

### 8.2 axios

Vue本身不支持发送AJAX请求，需要使用vue-resource、[axios](https://so.csdn.net/so/search?from=pc_blog_highlight&q=axios)等插件实现发送Ajax请求。 axios是一个基于Promise的HTTP请求客户端，用于发送HTTP请求，也是[vue](https://so.csdn.net/so/search?from=pc_blog_highlight&q=vue)2.0官方推荐的，同时不再对vue-resource进行更新和维护。

axios GitHub开源地址： https://github.com/axios/axios

#### 安装方法

* 使用CDN

```html
//二选其一
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script src="https://cdn.staticfile.org/axios/0.18.0/axios.min.js"></script>
```

* 使用命令安装

```shell
#npm
npm install axios

#bower 
brower install axios

#yarn 
yarn add axios
```



#### 基本使用

（1）**基本用法**

**axios([options])**

```js
send(){
    axios({
        methods:'get',
        url:"axios.json"
    }).then(function(resp){
        console.log(resp.data);
    }).catch(err => {
        console.log('请求失败：'+err);
    });
}
```



**（1）发送GET请求**

基本语法：`axios.get(url[,options]);`  支持两种传参方式：

* （1）通过url拼接参数传参：`axios.get(‘url?key=value&key1=val2’).then(reponse=>......)`
* （2）通过params选项传参：` axios.get(‘url’,{params:{key:value}}).then()`

```js
<div id="app">
    <table border="1">
        <thead>
        <tr>
            <th>姓名</th>
            <th>年龄</th>
            <th>地址</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(item) in list" :key="item.name">
            <td>{{ item.name }}</td>
            <td>{{ item.age }}</td>
            <td>{{ item.address }}</td>
        </tr>
        </tbody>
    </table>
</div>

<script type="text/javascript">

    const vm = new Vue({
        el: '#app',
        data: {
            list: [
                {
                    "name": "法外狂徒1",
                    "address": "本宇宙-拉尼亚凯亚超星系团-室女座星系团-本星系群-银河系-猎户座旋臂-太阳系-地球",
                    "age": 100
                }
            ],
            username: '',
            address: '',
            age: '',
            info:''
        },
        methods: {
            getUserInfoLists() {
                axios.get("http://127.0.0.1:8001/v1/demo/list")
                    .then((response) => {
                    //response时返回数据
                    console.log(response)
                    if (response.data.success) {
                        this.list=response.data.data;
                    }
                });
            }
        },
        created(){
            //在vue生命周期的created时请求数据
            this.getUserInfoLists();
        }
    })
</script>
```



**（2）发送POST请求**

基本语法：`axios.post(url,data[,options]);`

**axios默认发送post请求时，数据格式是Request Payload，并非我们常用的Form Data格式，所以参数必须以键值对形式传递，不能以json形式传参**。

支持三种传参方式：

* （1）自己拼接为键值对。
* （2）使用transformRequest，在请求发送前将请求数据进行转换。
* （3）如果使用模块化开发，可以使用qs模块进行转换。

```js
sendPost(){
    // axios.post('http://localhost:8080/v1/in/demo/user/add',"name=alice&age=16") 方式一：麻烦
    axios.post('http://localhost:8080/v1/in/demo/user/add',this.user,{
        transformRequest:[
            //转换数据格式
            function(data){
                let params = '';
                for(let index in data){
                    params+=index+'='+data[index]+'&';
                }
                return params;
            }
        ]
    })
    .then(resp =>{
        console.log(resp.data);
    }).catch(err =>{
        console.log('请求失败：'+err);
    });
}
```





## 九、Vue 过渡 & 动画

在 本教程的 第4.3 介绍了如何通过不同条件渲染页面上的元素，下面时一段使用 `v-show` 实现的代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> Vue 动画和过度 </title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>

<div id="app">
    <input type="button"  value="点我试试" @click="isDisable=!isDisable" />
    <h1 v-show="isDisable" >这是一段文本</h1>
</div>


<script>
    const vm=new Vue({
        el:'#app',
        data:{
            isDisable: true
        },
        methods:{

        }
    })
</script>
</body>
</html>
```



从实现效果可以看到元素的出现和消失都十分突兀，没有中间过度动画，容易给用户造成不好的使用体验，本节九基于vue提供的动画相关的功能实现对页面元素显示和消息动画的控制。

### 9.1 使用过度类名实现动画

Vue 在插入、更新或者移除 DOM 时，提供多种不同方式的应用过渡效果。Vue 提供了内置的过渡封装组件 `transition`，该组件用于包裹要实现过渡效果的组件。

<img src="http://image.easyblog.top/transition.png" style="width:80%;" />

上面是vue官网对过度动画原理的示意图，在进入/离开的过渡中，会有 6 个 class 切换。

1. `${name}-enter`：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。
2. `${name}-enter-active`：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡/动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。
3. `${name}-enter-to`：**2.1.8 版及以上**定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 `v-enter` 被移除)，在过渡/动画完成之后移除。
4. `${name}-leave`：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
5. `${name}-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
6. `${name}-leave-to`：**2.1.8 版及以上**定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 `v-leave` 被删除)，在过渡/动画完成之后移除。

对于这些在过渡中切换的类名来说，如果你使用一个没有名字的 `<transition>`，则默认这些类名的前缀就是 `v-` 。如果使用了 `<transition name="my-transition">`，那么 `v-enter` 会替换为 `my-transition-enter`。

总结：

* （1）`${name}-enter` 和 `${name}-leave-to` 分别表示元素进场之前和出场之后的状态，一般没有特殊要求时，定义样式的时候可以放到一组定义
* （2）`${name}-leave` 和 `${name}-enter-to`  分别表示元素进场之后和出场之前的状态，这两没有特殊要求也可以在定义样式的时候写的一起
* （3）`${name}-enter-active` 和 `${name}-leave-active` 就是进场/离场时的动画效果

如下是一个使用默认class实现的左右淡入淡出的过度动画：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> Vue 动画和过度 </title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        /**
        定义元素进场之前 以及 元素出场 之后的状态
         */
        .v-enter,
        .v-leave-to{
            opacity: 0;
            transform: translateX(200px);
        }

        /**
        定义元素进场/出场时的动画效果
         */
        .v-enter-active,
        .v-leave-active{
            transition: all 0.4s ease;
        }
    </style>
</head>
<body>

<div id="app">
    <input type="button"  value="点我试试" @click="isDisable=!isDisable" />
    <transition>
        <h1 v-show="isDisable" >这是一段文本</h1>
    </transition>
</div>


<script>
    const vm=new Vue({
        el:'#app',
        data:{
            isDisable: true
        },
        methods:{

        }
    })
</script>
</body>
</html>
```



### 9.2 自定义过渡的类名

9.1 中了解了vue动画的基本使用方式，就是首先使用 `transition` 组件将需要动画的元素包裹起来，然后定义过度动画的 class 即可实现元素的动画效果。但是当页面上有多个元素需要不同的动画效果时，比如：有些元素需要左右飞入飞出、有些元素需要上下飞入飞出...... 为实现这些需要，只靠vue默认的class时不够的，需要我们**自定义过度类名实现不同元素，不同动画效果**。

实现方式也很简单，首先就是在使用 `transition` 组件的时候，给他的 `name` 属性自定义一个类名前缀名称 prefix，之后在给 prefix-enter、prefix-enter、prefix-leave、prefix-leave-to....这些类名，如同使用默认过度类名那样定义这些自定义过度类名的样式即可，如下是一个使用示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> Vue 动画和过度 </title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        /**
        定义元素进场之前 以及 元素出场 之后的状态
         */
        .v-enter,
        .v-leave-to{
            opacity: 0;
            transform: translateX(200px);
        }

        /**
        定义元素进场/出场时的动画效果
         */
        .v-enter-active,
        .v-leave-active{
            transition: all 0.4s ease;
        }

        .fade-enter,
        .fade-leave-to{
            opacity: 0;
            transform: translateY(300px);
        }

        .fade-leave-active,
        .fade-enter-active{
            transition: all 1s ease;
        }
    </style>
</head>
<body>

<div id="app">
    <input type="button"  value="点我试试" @click="isDisable=!isDisable" />
    <transition>
        <h1 v-show="isDisable" >这是左右飞入飞出文本</h1>
    </transition>

    <transition name="fade">
        <h1 v-show="isDisable" >这是上下飞入飞出文本</h1>
    </transition>
</div>


<script>
    const vm=new Vue({
        el:'#app',
        data:{
            isDisable: true
        },
        methods:{

        }
    })
</script>
</body>
</html>
```



### 9.3 使用第三方CSS类库实现动画

上面我们分别学会了使用默认过度类和自定义过度类实现不同粒度下对元素过度动画的控制，其实有时候有些动画效果已经有第三方写好了包，我们只需要导入这些包即可轻松实现一些高难度的动画，从而提高生产效率，vue的 `transition` 组件也支持直接使用第三方动画来实现过度动画，下面将以业内有名的动画库 [animate.css](https://animate.style/) 为例来进行试验：

（1）首先需要导入 `animate.css` 有关的依赖包：

```shell
#使用CDN
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.2/animate.min.css">
#npm
npm install animate.css --save
#yarn
yarn add animate.css
```

（2）示例：实现从自上而下的飞入飞出效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> Vue 动画和过度 </title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.2/animate.min.css">
</head>
<body>

<div id="app">
    <input type="button"  value="点我试试" @click="isDisable=!isDisable" />
    <transition enter-active-class="animated bounceInDown" leave-active-class="animated bounceOutDown">
        <h1 v-show="isDisable" >这是bounceInDown 和 bounceOutDown 效果文本</h1>
    </transition>
</div>


<script>
    const vm=new Vue({
        el:'#app',
        data:{
            isDisable: true
        }
    })
</script>
</body>
</html>
```



### 9.4 JavaScript 钩子实现半场动画

有时候我们只想要入场动画的特效不需要离场动画的特效，或者反过来，这是依靠上面使用类名的方式就无法更加精准的控制元素的动画特效了，好在 vue 提供了` JavaScript 钩子` 来供我们实现半场动画，顾名思义，就是只有一半生命周期的动画。

vue 允许我们在使用 `transition` 元素的时候绑定不同阶段的事件（钩子）来实现更细粒度的动画控制，如下是vue允许我们绑定在一个transition上的所有事件（钩子）：

```js
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
```

> 其实这东西可以理解为动画的生命周期，vue会在动画的不同周期上执行不同的钩子函数，我们所需要做的就是在transition上注册需要的事件，并且像实现普通事件方法一样实现这些方法。



下面是一个实现类似点击添加购物车后小球从商品位置跳转到屏幕底部购物车位置的效果动画：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> Vue 动画和过度 </title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .ball{
            width: 15px;
            height: 15px;
            border-radius: 50%;
            background-color: red;
        }
    </style>
</head>
<body>

<div id="app">
    <input type="button"  value="立即购买" @click="isDisable=!isDisable" />
    <transition
    @before-enter="beforeEnter"
    @enter="enter"
    @after-enter="afterEnter">
        <div class="ball" v-show="isDisable" ></div>
    </transition>
</div>


<script>
    const vm=new Vue({
        el:'#app',
        data:{
            isDisable: false
        },
        methods: {
            //动画钩子的第一个参数参数el会在vue调用的时候闯传入要执行动画的DOM元素
            beforeEnter(el){
                //定义元素动画开始前的起始位置
                el.style.transform="translate(0,0)";
            },
            enter(el,done){
               //动画开始之后的动作，也就是动画的动作
                el.offsetHeight;  //强制刷新浏览器显示动画效果
                el.style.transform="translate(150px,450px)";
                el.style.transition="all 1s ease";

                //会在动画结束后立即调用 after-enter 事件中定义的逻辑
                done();
            },
            afterEnter(el){
                //动画结束之后的动作，这里需要让小球消失
               this.isDisable=!this.isDisable;
            }
        }
    })
</script>
</body>
</html>
```



### 9.5 使用 transition-group 实现列表动画

有时候我们同时渲染整个列表使用动画，比如使用 `v-for`？在这种场景中，vue 提供了 `<transition-group>` 组件。在我们深入例子之前，先了解关于这个组件的几个特点：

- 不同于 `<transition>`，它会以一个真实元素呈现：默认为一个 `<span>`。你也可以通过 `tag` 属性更换为其他元素。
- [过渡模式](https://cn.vuejs.org/v2/guide/transitions.html#过渡模式)不可用，因为我们不再相互切换特有的元素。
- 内部元素**总是需要**提供唯一的 `key` 属性值。
- CSS 过渡的类将会应用在内部的元素中，而不是这个组/容器本身。

除了上面这些特点，`<transition-group>` 组件还有一个特殊之处。不仅可以实现进入和离开动画，还可以改变定位。要使用这个新功能只需使用新增的 **`v-move` 过度类**，它会在元素的改变定位的过程中应用。像之前的类名一样，可以通过 `name` 属性来自定义前缀，也可以通过 `move-class` 属性手动设置。

`v-move` 对于设置过渡的切换时机和过渡曲线非常有用，如下是一个基于列表实现各种动画效果：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> Vue 动画和过度 </title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        li {
            border: 1px black dashed;
            margin: 5px;
            line-height: 40px;
            width: 100%;
        }

        li:hover {
            background-color: lightgray;
            transition: all 0.4s linear;
        }

        .v-enter,
        .v-leave-to {
            opacity: 0;
            transform: translateY(80px);
        }

        .v-enter-active,
        .v-leave-active {
            transition: all 0.8s ease-in-out;
        }

        /**
        可以实现列表后续内容丝滑上移的动画效果,这个类的前缀也可以自定义
         */
        .v-move {
            transition: all 0.5s ease-in-out;
        }

        .v-leave-active {
            position: absolute;
        }
    </style>
</head>
<body>

<div id="app">


    <label for="name">
        Name:
        <input type="text" v-model:value="name" id="name"/>
    </label>
    <label for="name">
        Age:
        <input type="text" v-model:value="age" id="age"/>
    </label>
    <label for="name">
        Address:
        <input type="text" v-model:value="address" id="address"/>
    </label>

    <input type="button" value="增加" @click="add">

   <!--
    appear: 定义列表页面加载时的入场动画
    tag: 定义transition-group的真实元素为ul
   -->
    <transition-group appear tag="ul">
        <li v-for="(item,index) in users" :key="item.name" @click="del(index)">
            {{ item.name }}---{{ item.age }}---{{ item.address }}
        </li>
    </transition-group>

</div>


<script>
    const vm = new Vue({
        el: '#app',
        data: {
            name: '',
            age: '',
            address: '',
            users: [
                {
                    "name": "法外狂徒1",
                    "address": "本宇宙-拉尼亚凯亚超星系团-室女座星系团-本星系群-银河系-猎户座旋臂-太阳系-地球",
                    "age": 200
                },
                {
                    "name": "法外狂徒2",
                    "address": "本宇宙-拉尼亚凯亚超星系团-室女座星系团-本星系群-银河系-猎户座旋臂-太阳系-地球",
                    "age": 300
                },
                {
                    "name": "法外狂徒3",
                    "address": "本宇宙-拉尼亚凯亚超星系团-室女座星系团-本星系群-银河系-猎户座旋臂-太阳系-地球",
                    "age": 400
                },
                {
                    "name": "法外狂徒4",
                    "address": "本宇宙-拉尼亚凯亚超星系团-室女座星系团-本星系群-银河系-猎户座旋臂-太阳系-地球",
                    "age": 500
                }
            ]
        },
        methods: {
            add() {
                this.users.push({
                    name: this.name,
                    age: this.age,
                    address: this.address
                })
            },
            del(index) {
                this.users.splice(index, 1);
            }
        }
    })
</script>
</body>
</html>
```



## 十、Vue 组件

组件系统是Vue.js其中一个重要的概念，它提供了一种抽象，让我们可以使用独立可复用的小组件来构建大型应用，任意类型的应用界面都可以抽象为一个组件树。

![](http://image.easyblog.top/4804567-7cfaa617959198be.png)

**组件 (Component)** 是 Vue.js 最强大的功能之一。组件可以扩展 HTML 元素，封装可重用的代码。在较高层面上，组件是自定义元素，Vue.js 的编译器为它添加特殊功能。在有些情况下，组件也可以表现为用 is 特性进行了扩展的原生 HTML 元素。所有的 Vue 组件同时也都是 Vue 的实例，所以可接受相同的选项对象 (除了一些根级特有的选项) 并提供相同的生命周期钩子



### 10.1 注册组件

**（1）使用 Vue.component 和 Vue.extend 定义组件**

```html
<script>

    //自定义组件方式1：Vue.component 配合 Vue.extends
    Vue.component("login",Vue.extend({
        template: '<form>Username:<input type="text" name="username"><br>Password:<input type="password" name="password"><br><input type="button" value="登录"></form>'}));

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {}
    });
</script>


<div id="app">
    <!--使用自定义组件-->
    <login></login>
</div>
```



**（2）直接使用Vue.component定义组件**

方法（1）中使用`Vue.extend` 定义组件模板的方式显得有些麻烦，其实vue支持我们直接使用一个`{}`来定义组件的模板

```html
<script>

    //自定义组件方式2：
    Vue.component("login-ch",{
        template: '<form>用户名:<input type="text" name="username"><br>密码:<input type="password" name="password"><br><input type="button" value="登录"></form>'
    })
    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {}
    });
</script>


<div id="app">
    <!--使用自定义组件-->
    <login-ch></login-ch>
</div>
```



**（3）使用 `<template>` 标签定义组件模板**

其实（1）（2）两种定义组件的方式大同小异，都是直接在template属性中写入HTML模板内容，这点在项目中可能不是很好维护，而且尤其第（2）种在定义组件的时候还没有代码提示，非常不方便，于是vue又提供了一个新组件 `template` ，我们可以在一个vue实例作用域之外使用该组件，我们需要做的就是将组件模板HTML代码使用 `template` 包裹起来，如下是一个示例：

```html
<!--定义Template 使用-->
<template id="login_form">
    <form>
        <label for="username">
            Username
            <input type="text" id="username" name="username">
        </label>
        <label for="password">
            Username
            <input type="text" id="password" name="password">
        </label>
        <input type="button"  value="login">
    </form>
</template>

<script>
    //自定义组件的第三种方式
    Vue.component("login-form",{
        template: "#login_form"
    })
    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {}
    });
</script>

<div id="app">
    <!--使用组件-->
    <login-form></login-form> 
</div>
```



### 10.2 定义私有组件

上面使用 `Vue.component(name,{})` 定义的组件我们成为全局组件，就是在项目全局都可以使用的，有时候我们只想某些组件在某个vue实例下有效就可以了，这就需要用到vue实例的 `components` 属性。下面时一个定义私有组件的示例：

```html
<!--定义组件模板-->
<template id="title">
    <h1>XXX网站</h1>
</template>

<script>
    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        components: {
            //组件名 title
            myTitle: {
                //组件模板
                template: '#title'
            }
        }
    });
</script>

<div id="app">
    <!--使用组件-->
    <my-title></my-title>
</div>
```



### 10.3 组件的属性

10.1 和 10.2 展示了如何使用vue定义全局和私有组件，其实重点就是及使用了 `Vue.componet()` 的 `template` 属性，其实vue还支持非常多的属性，比如 `data`、`methods`  等属性，作用和vue实例中的 `data`、`methods`  等属性一样，但是组件中的`data`属性写法有点不同，**一个组件的 `data` 选项必须是一个函数，并且必须返回一个对象**。下面是一个计数器组件示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>自定义计数器</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div id="app">

<counter></counter>
</div>

<template id="counter">
    <div>
        <input type="button" @click="addCount" value="+1">

        <p>按钮点击次数：{{ count }}</p>
    </div>
</template>

<script>
    Vue.component('counter', {
        template: '#counter',
        data: function () {
            //这里必须返回一个对象
            return {count: 0};
        },
        //也可以给组件定义方法
        methods: {
            addCount(){
                this.count++;
            }
        }
    })

    let vm = new Vue({
        el: '#app',
        data: {},
        methods: {}
    });
</script>
</body>
</html>
```



### 10.4 组件的切换和组件动画

#### 10.4.1 基于v-if、v-show实现组件切换

这种控制组建的显示和消息的原理就和使用`v-if`、`v-show` 控制普通元素的原理时一直的，都是通过设置一个标记值`flag` 当flag成立时显示A，反之显示B。所以重点在于如何正确的控制标记值`flag` ，如下是一个通过点击登录或注册按钮而显示对应组件的示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>组件切换动画</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div id="app">
    <a href="#" @click.prevent="flag=true">登录</a>
    <a href="#" @click.prevent="flag=false">注册</a>
    <!--通过v-if控制显示那个组件 这里使用v-show同理-->
    <login v-if="flag"></login>
    <register v-else="flag"></register>
</div>
<!--定义login组件-->
<template id="login">
    <div>
        <form>
            <label for="login-username">
                Username
                <input type="text" id="login-username" name="username">
            </label>
            <label for="login-password">
                Username
                <input type="text" id="login-password" name="password">
            </label>
            <input type="button" value="login">
        </form>
    </div>
</template>

<!--定义注册组件-->
<template id="register">
    <div>
        <form>
            <label for="register-username">
                Username
                <input type="text" id="register-username" name="username">
            </label>
            <label for="register-password">
                Username
                <input type="text" id="register-password" name="password">
            </label>
            <input type="button" value="register">
        </form>
    </div>
</template>

<script>
    Vue.component('login', {
        template: '#login',
    });

    Vue.component('register', {
        template: '#register',
    });

    let vm = new Vue({
        el: '#app',
        data: {
            flag: true
        }
    });
</script>
</body>
</html>
```



#### 10.4.2 使用 `<component>`控制组件切换

除了10.4.1这种切换组件的方式，vue还提了一和新的组件 `<component>`，它可以专门用来切换处理组件切换。主要依赖它的 `is` 属性，使用的时候，我们只需要给他传过去正确的组件id，然后该组件就可以正确的将对应组件渲染出来，如下示例基于10.4.1中代码的`<component>` 版本：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>组件切换动画</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div id="app">
    <a href="#" @click.prevent="componentName='login'">登录</a>
    <a href="#" @click.prevent="componentName='register'">注册</a>
    <!--使用component组件 实现组件的切换-->
    <component  :is="componentName"></component>
</div>

<template id="login">
    <div>
        <form>
            <label for="login-username">
                Username
                <input type="text" id="login-username" name="username">
            </label>
            <label for="login-password">
                Username
                <input type="text" id="login-password" name="password">
            </label>
            <input type="button" value="login">
        </form>
    </div>
</template>

<template id="register">
    <div>
        <form>
            <label for="register-username">
                Username
                <input type="text" id="register-username" name="username">
            </label>
            <label for="register-password">
                Username
                <input type="text" id="register-password" name="password">
            </label>
            <input type="button" value="register">
        </form>
    </div>
</template>

<script>
    Vue.component('login', {
        template: '#login',
    });

    Vue.component('register', {
        template: '#register',
    });

    let vm = new Vue({
        el: '#app',
        data: {
            //默认显示login
            componentName: 'login'
        }
    });
</script>
</body>
</html>
```



#### 10.4.3 组件切换设置动画

其实只要了解vue动画相关的知识，给组件加入动画的原理是一样的，使用 `transition` 也可以实现组件动画，如下是一个示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>组件切换动画</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <style>
        .v-enter,
        .v-leave-to{
            opacity: 0;
            transform: translateY(150px);
        }

        .v-enter-active,
        .v-leave-active{
            transition: all 0.5s ease;
        }
    </style>
</head>
<body>
<div id="app">
    <a href="#" @click.prevent="componentName='login'">登录</a>
    <a href="#" @click.prevent="componentName='register'">注册</a>
    <!--使用transition可以轻松实现组件的切换
      mode 有两个取值：out-in  和 in-out  分别表示组件时先出后进还是先进后出
    -->
    <transition mode="out-in">
       <component  :is="componentName"></component>
    </transition>
</div>

<template id="login">
    <div>
        <form>
            <label for="login-username">
                Username
                <input type="text" id="login-username" name="username">
            </label>
            <label for="login-password">
                Username
                <input type="text" id="login-password" name="password">
            </label>
            <input type="button" value="login">
        </form>
    </div>
</template>

<template id="register">
    <div>
        <form>
            <label for="register-username">
                Username
                <input type="text" id="register-username" name="username">
            </label>
            <label for="register-password">
                Username
                <input type="text" id="register-password" name="password">
            </label>
            <input type="button" value="register">
        </form>
    </div>
</template>

<script>
    Vue.component('login', {
        template: '#login',
    });

    Vue.component('register', {
        template: '#register',
    });

    let vm = new Vue({
        el: '#app',
        data: {
            //默认显示login
            componentName: 'login'
        }
    });
</script>
</body>
</html>
```



### 10.5 父子组件通信

在vue中，父子组件的关系可以总结为`prop`向下传递，`事件`向上传递。父组件通过`prop`给子组件下发数据，子组件通过`事件`给父组件发送信息。

![img](http://image.easyblog.top/2891127-591b88f49fb05f19.png)



#### 10.5.1 父组件向子组件传参

父组件到子组件的通信主要为：子组件接受使用父组件的数据，这里的数据包括属性和方法（String, Number, Boolean, Object, Array, Function）。vue提倡**单项数据流**，因此在通常情况下都是父组件传递数据给子组件使用，子组件触发父组件的事件，并传递给父组件所需要的参数

**（1）通过 `props` 属性传递数据 (推荐)**

父子通信中最常见的数据传递方式就是通过 `props` 属性传递数据，就好像方法的传参一样，父组件调用子组件并传入数据，子组件接受到父组件传递的数据进行验证使用

**props 可以是数组或对象，用于接收来自父组件的数据**。props 可以是简单的数组，或者使用对象作为替代，对象允许配置高级选项，如类型检测、自定义校验和设置默认值。下面是一个使用 `proops` 属性实现的父组件向子组件传参的示例：

```html
<div id="app">
  <message :parent-msg="msg"></message>
</div>


<template id="message">
    <div>
        <p>我是子组件，我从父组件获得参数：{{parentMsg}}</p>
    </div>
</template>

<script>

    Vue.component("message",{
        template: '#message',
        props:['parentMsg']
    });

    var vm = new Vue({
        el: '#app',
        data: {
            msg: 'Hello child!!!!'
        },
        methods: {}
    });
</script>
```

**要点**：

* 1）在需要父组件传参的子组件标签上使用`v-bind` 或 `:`  自定义属性绑定到父组件的参数上，比如这里的 `msg`
* 2）在子组件中使用 `props` 属性定义自定义属性的名称，并在子组件需要的地方使用该属性，比如这里的 `parentMsg`



#### 10.5.2 子组件向父组件传参

子组件到父组件的通讯主要为父组件如何接受子组件之中的数据。这里的数据包括属性和方法（String, Number, Boolean, Object, Array, Function）



**（1）通过 事件机制 配合 `$emit` 通过方法传参实现子组件向父组件传参**

与父组件到子组件通讯中的 `$on` 配和使用，可以向父组件中触发的方法传递参数供父组件使用

```html
<div id="app">
    <message @func="show" :parent-msg="msg"></message>
    <p>获取到子组件的参数: {{childMsg}}</p>
</div>


<template id="message">
    <div>
        <p>我是子组件，我从父组件获得参数：{{ parentMsg }}</p>
        <input type="button" value="点击给父组件发送参数" @click="sendMsgToParent">
    </div>
</template>

<script>

    Vue.component("message", {
        template: '#message',
        props: ['parentMsg'],
        data: function () {
            return {
                childMsg: {name: '张三', message: 'Hello parent!'}
            }
        },
        methods: {
            sendMsgToParent() {
                //通过 事件机制 配合 $emit 通过方法传参实现父子组件传参,
                // $emit 方法从第二个行参开始都可以给父组件方法传参
                this.$emit('func',this.childMsg);
            }
        }
    });

    var vm = new Vue({
        el: '#app',
        data: {
            msg: 'Hello child!!!!',
            childMsg:''
        },
        methods: {
            show(data) {
                console.log("调用了父组件的show----,参数："+data);
                //方法执行的时候将子组件传过来的参数保存到父组件的data就实现了子组件到父组件的传参
                this.childMsg=data;
            }
        }
    });
</script>
```

**要点**：

* 1）使用事件机制自定义事件绑定到父组件的某个方法，比如这里的子组件的自定义 `func` 事件绑定到了 父组件的 `show` 方法
* 2）在子组件中使用 `this.$emit('event')` 调用自定义事件，比如这里的 `this.$emit('func')` 就表示触发这个自定义 `func` 事件，然后这个事件就会调用后面的事件处理方法 `show`
* 3）在使用 `this.$emit('event')` 调用的时可以在第二个形参开始为父组件传递参数



**（2）通过`$refs`获取引用对象，进而获取参数**

**示例**：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vue父子组件通信</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div id="app">
    <message ref="message"></message>
    <p>从子组件获得信息：{{childMsg}}</p>
    <input type="button" @click="show" value="获取参数值">
</div>


<template id="message">
    <div>
        <p>我是一个子组件</p>
    </div>
</template>

<script>

    Vue.component("message", {
        template: '#message',
        data: function () {
            return {
                childMsg: {name: '张三', message: 'Hello parent!'}
            }
        },
        methods: {
            show(){
                console.log("调用了子组件的show方法")
            }
        }
    });

    var vm = new Vue({
        el: '#app',
        data: {
            msg: 'Hello child!!!!',
            childMsg: {}
        },
        methods: {
            show() {
                console.log("调用了父组件的show----");
                //从引用对象中获取子组件的属性值
                console.log(this.$refs.message)
                this.childMsg=this.$refs.message.childMsg;
                //父组件调用子组件的方法
                this.$refs.message.show();

            }
        }
    });
</script>
</body>
</html>
```

**要点**：

* 1）在组件上使用 `ref` 属性指定主键引用的名称，比如这的 `ref='message'`
* 2）直接在父组件中使用 `this.$refs.{ref_name}` 就可以获取到对应组件的对象，有了对象自然就可以操作对象里面各种数据，方法了。另外，此方法也可以用于代替 js 传统的通过 `documetn.getElementByXxxx()` 直接操作DOM的方式获取元素对象。



## 十一、Vue路由



### 11.1 vue-router是什么

这里的路由并不是指我们平时所说的硬件路由器，**这里的路由就是SPA（单页应用）的路径管理器**。再通俗的说，vue-router就是WebApp的**链接路径管理系统**。

vue-router是Vue.js官方的路由插件，它和vue.js是深度集成的，适合用于构建单页面应用。vue的单页面应用是基于路由和组件的，路由用于设定访问路径，并将路径和组件映射起来。传统的页面应用，是用一些超链接来实现页面切换和跳转的。在vue-router单页面应用中，则是路径之间的切换，也就是组件的切换。**路由模块的本质 就是建立起url和页面之间的映射关系**。

至于我们为啥不能用a标签，这是因为用Vue做的都是单页应用，就相当于只有一个主`index.html`页面，所以你写的`<a></a>`标签是不起作用的，你必须使用vue-router来进行管理。

`

### 11.2 vue-router实现原理

SPA(single page application):单一页面应用程序，只有一个完整的页面；它在加载页面时，不会加载整个页面，而是只更新某个指定的容器中内容。**单页面应用(SPA)的核心之一是: 更新视图而不重新请求页面**;vue-router在实现单页面前端路由时，提供了两种方式：Hash模式和History模式；根据mode参数来决定采用哪一种方式。

#### 1、Hash模式：

**vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。** hash（#）是URL 的锚点，代表的是网页中的一个位置，单单改变#后的部分，浏览器只会滚动到相应位置，不会重新加载网页，也就是说 #是用来指导浏览器动作的，对服务器端完全无用，HTTP请求中也不会不包括#；同时每一次改变#后的部分，都会在浏览器的访问历史中增加一个记录，使用”后退”按钮，就可以回到上一个位置；所以说**Hash模式通过锚点值的改变，根据不同的值，渲染指定DOM位置的不同数据**。

```js
var router = new VueRouter({
  mode: 'hash',  //选择hash模式，默认就是hash模式
  routes: [...]
 });    
```



#### 2、History模式：

由于hash模式会在url中自带#，如果不想要很丑的 hash，我们可以用路由的 history 模式，只需要在配置路由规则时，加入"mode: 'history'",这种模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

```js
var router = new VueRouter({
  mode: 'history',  //选择history模式
  routes: [...]
 });          
```



### 11.3 vue-router基本使用

#### 11.3.1 安装vue-router

**（1）使用CDN**

```html
https://unpkg.com/vue-router@2.0.0/dist/vue-router.js
```



**（2）npm**

```html
npm install vue-router
```



#### 11.3.2 vue-router 动态路由匹配

在vue中实现路由还是相对简单的。因为我们页面中所有内容都是组件化的，我们只要把路径和组件对应起来就可以了，然后在页面中把组件渲染出来。

**（1） js 中配置路由**

首先要定义route,  一条路由的实现。它是一个对象，由两个部分组成： `path` 和 `component`，` path` 指路径，`component` 指的是组件,这里的组件一定要是组件模板对象，不能是使用`Vue.component` 创建的组件对象引用。如下是一个路由的配置示例：

```js
//login组件模板对象
var login={
    template: '<h1>登录组件</h1>'
}

//register组件模板对象
var register={
    template: '<h1>注册组件</h1>'
}

var rotuer =new VueRotuer({
    routes:[
        //可以配置多条路由规则
        //path 表示路由规则，在url地址上就是#后面的路径
        //component 表示组件模板对象，不能是组件对象的引用
        {path:'/login',component: login},
        {path:'/lregister',component: register}
    ]
})
```

　配置完成后，需要把 `rotuer` 实例注入到 vue 根实例中,就可以使用路由了：

```js
var vm=new Vue({
    el:'#app',
    data:{},
    methods:{}
    rotuer: rotuer //配置路由实例
})
```



**（2）页面实现（HTML模版）**

vue-router提供了两个标签 `<router-link>` 和 `<router-view>` 来对应点击和显示部分。

* `<router-link>` 可以用于定义页面中点击的部分，可以代替 `<a> ` 标签； `<router-link> ` 还有一个非常重要的属性 to，定义点击之后，要到哪里去， 如：`<router-link  to='/home'>Home</router-link>`，表示点击之后会去 home 组件，此外 `<router-link>` 会被vue默认渲染成 `<a>` 标签，如果想换成其他标签，可以使用 `tag` 属性，比如切换想渲染成 `<span>` 标签，可以这么做：`<router-link to='/login' tag='span'>Login</router-link>`
* `<router-view> ` 可以用于显示部分，就是点击后，匹配的内容显示在什么地方，作用相当于一个占位符。

如下是路由的基本完整使用示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue-router</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
</head>
<body>

<div id="app">

    <router-link to="/login">登录</router-link>
    <router-link to="/register">注册</router-link>
    <!--用于定义组件显示的位置，会被vue模板替换，在这里只是相当于一个占位符-->
    <router-view></router-view>

</div>

<script type="text/javascript">

    //组件模板对象
    var login =  {
        template: '<h1>登录组件</h1>'
    }

    var register = {
        template: '<h1>注册组件</h1>'
    }

    var routerObj = new VueRouter({
        routes: [
            //定义路由规则
            //path: 路由规则
            //component 只能是组件模板对象，不能是组件的引用名
            {path: '/login', component: login},
            {path: '/register', component: register}
        ]
    });

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router: routerObj
    });
</script>
</body>
</html>
```



### 11.4 路由的重定向 redirect 

```js
 var routerObj = new VueRouter({                  
     routes: [//定义路由规则 
         //重定向 将/路径重定向到/login
         {path:'/',redirect: '/login'},           
         //path: 路由规则                             
         //component 只能是组件模板对象，不能是组件的引用名          
         {path: '/login', component: login},      
         {path: '/register', component: register} 
     ]                                            
 });                                              
```



### 11.5 路由active-class

#### 11.5.1 使用vue-router默认激活类定义激活样式

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211225142002.png)

查看`<router-link>`标签经过vue渲染后的页面元素，发现vue会默认给应用上两个类 `router-link-active` 和 `router-link-exact-active` 这个类我们可以用于定义给点击的按钮定义一些激活样式。如下是一个示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue-router</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        /*
        定义激活属性
        */
        .router-link-active{
            color: coral;
            font-size: 20px;
            font-weight: 800;
        }
    </style>
</head>
<body>

<div id="app">

    <router-link to="/login">登录</router-link>
    <router-link to="/register">注册</router-link>

   <router-view></router-view>

</div>

<script type="text/javascript">

    //组件模板对象
    var login =  {
        template: '<h1>登录组件</h1>'
    }

    var register = {
        template: '<h1>注册组件</h1>'
    }

    var routerObj = new VueRouter({
        routes: [
            //定义路由规则
            {path:'/',redirect: '/login'},
            //path: 路由规则
            //component 只能是组件模板对象，不能是组件的引用名
            {path: '/login', component: login},
            {path: '/register', component: register}
        ]
    });

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router: routerObj
    });
</script>
</body>
</html>
```



#### 11.5.2 使用 vue 提供的 `linkActiveClass` 属性使用第三方样式

除了上面使用默认 `router-link-active`  定义激活样式的方式，vue也支持使用直接在实例化路由属性的时候使用 `linkActiveClass` 第三方样式。如下是一个示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue-router</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        //第三方或自定义css样式
        .my-active{
            color: coral;
            font-size: 20px;
            font-weight: 800;
        }
    </style>
</head>
<body>

<div id="app">

    <router-link to="/login">登录</router-link>
    <router-link to="/register">注册</router-link>

   <router-view></router-view>

</div>

<script type="text/javascript">

    //组件模板对象
    var login =  {
        template: '<h1>登录组件</h1>'
    }

    var register = {
        template: '<h1>注册组件</h1>'
    }

    var routerObj = new VueRouter({
        routes: [
            //定义路由规则
            {path:'/',redirect: '/login'},
            //path: 路由规则
            //component 只能是组件模板对象，不能是组件的引用名
            {path: '/login', component: login},
            {path: '/register', component: register}
        ],
        //使用 linkActiveClass 属性指定第三方或自定义激活样式
        linkActiveClass: 'my-active'
    });

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router: routerObj
    });
</script>
</body>
</html>
```



### 11.6 路由传参

#### 11.6.1 使用query方法传参

父组件：使用path来匹配路由，然后在子组件中通过query来获取父组件传递的参数

这种情况下， query传递的参数会显示在url后面，并且路由匹配规则不要用改变

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue-router</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        .my-active {
            color: coral;
            font-size: 20px;
            font-weight: 800;
        }

        .v-enter, .v-leave-to {
            opacity: 0;
            transform: translateX(100px);
        }

        .v-enter-active,
        .v-leave-active {
            transition: all 0.5s ease;
        }
    </style>
</head>
<body>

<div id="app">

    <router-link to="/home">首页</router-link>
    <!--直接在url路径上传递参数-->
    <router-link to="/login?username=frank.huang&password=123456">登录</router-link>
    <router-link to="/register">注册</router-link>

    <transition mode="out-in">
        <router-view></router-view>
    </transition>

</div>

<template id="login">
    <div>
        <h1>登录组件</h1>
        <span>从父组件获得参数：username:{{username}}---pasword:{{password}}</span>
    </div>
</template>

<script type="text/javascript">

    //组件模板对象
    var login = {
        template: '#login',
        data(){
          return {
             username: '',
             password: ''
          };
        },
        created() {
            //组件钩子函数,组件创建的时候通过this.$route.query.[参数名]从url中获取参数
            this.username = this.$route.query.username;
            this.password = this.$route.query.password;
        }
    }

    var register = {
        template: '<h1>注册组件</h1>'
    }

    var home = {
        template: '<h1>首页组件</h1>'
    }

    var routerObj = new VueRouter({
        routes: [
            //定义路由规则
            {path: '/', redirect: '/home'},
            //path: 路由规则
            //component 只能是组件模板对象，不能是组件的引用名
            {path: '/home', component: home},
            {path: '/login', component: login},
            {path: '/register', component: register}
        ],
        linkActiveClass: 'my-active'
    });

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router: routerObj
    });
</script>
</body>
</html>
```



#### 11.6.2 使用params方法传参

父组件中：通过路由属性中的name来确定匹配的路由，通过params来传递参数。

使用这种方式传参，路由 path 需要对应改变，需要在路径中将需要带入子组件的参数写到路径下，而不是请求参数中，比如：`/login/:username`、`/login/:password` ，这种写法是不会正常工作的：`/login?:username`。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue-router</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        .my-active {
            color: coral;
            font-size: 20px;
            font-weight: 800;
        }

        .v-enter, .v-leave-to {
            opacity: 0;
            transform: translateX(100px);
        }

        .v-enter-active,
        .v-leave-active {
            transition: all 0.5s ease;
        }
    </style>
</head>
<body>

<div id="app">

    <router-link to="/home">首页</router-link>
    <router-link to="/login/frank.huang/123456">登录</router-link>
    <router-link to="/register">注册</router-link>

    <transition mode="out-in">
        <router-view></router-view>
    </transition>

</div>

<template id="login">
    <div>
        <h1>登录组件</h1>
        <span>从父组件获得参数：username:{{username}}---pasword:{{password}}</span>
    </div>
</template>

<script type="text/javascript">

    //组件模板对象
    var login = {
        template: '#login',
        data(){
          return {
             username: '',
             password: ''
          };
        },
        created() {
            //组件钩子函数,组件创建的时候从url中获取参数
            this.username = this.$route.params.username;
            this.password = this.$route.params.password;
        }
    }

    var register = {
        template: '<h1>注册组件</h1>'
    }

    var home = {
        template: '<h1>首页组件</h1>'
    }

    var routerObj = new VueRouter({
        routes: [
            //定义路由规则
            {path: '/', redirect: '/home'},
            //path: 路由规则
            //component 只能是组件模板对象，不能是组件的引用名
            {path: '/home', component: home},
            {path: '/login/:username/:password', component: login},
            {path: '/register', component: register}
        ],
        linkActiveClass: 'my-active'
    });

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router: routerObj
    });
</script>
</body>
</html>
```



### 11.7 路由嵌套（子路由）

实际生活中的应用界面，通常由多层嵌套的组件组合而成。同样地，URL 中各段动态路径也按某种结构对应嵌套的各层组件，例如：

```text
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  ------------->  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

这里的 `/user/foo/profile` 和 `/user/foo/posts` 都同是 User 父组件组件下的两个不同子组件，vue中借助 `vue-router`，使用嵌套路由配置，就可以很简单地表达这种关系。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue-router</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        .my-active {
            color: coral;
            font-size: 20px;
            font-weight: 800;
        }

        .v-enter, .v-leave-to {
            opacity: 0;
            transform: translateX(100px);
        }

        .v-enter-active,
        .v-leave-active {
            transition: all 0.5s ease;
        }
    </style>
</head>
<body>

<div id="app">

    <router-link to="/user">首页</router-link>

    <transition mode="out-in">
        <router-view></router-view>
    </transition>

</div>

<template id="user">
    <div>
        <h1>这是User组件</h1>
        <router-link to="/user/profile">profile</router-link>
        <router-link to="/user/posts">posts</router-link>
        <!--在子组件中再定义组件-->
        <transition mode="out-in">
            <router-view></router-view>
        </transition>
    </div>
</template>

<script type="text/javascript">

    //组件模板对象
    var user = {
        template: '#user'
    }

    var profile = {
        template: '<h1>profile组件</h1>'
    }

    var posts = {
        template: '<h1>posts组件</h1>'
    }

    var routerObj = new VueRouter({
        routes: [
            //定义路由规则
            //path: 路由规则
            //component 只能是组件模板对象，不能是组件的引用名
            //children 定义子路由，也就是嵌套路由
            {
                path: '/user',
                component: user,
                children: [
                    //和基本路由定义方法类似
                    {path: 'profile', component: profile},
                    {path: 'posts', component: posts}
                ]
            }
        ],
        linkActiveClass: 'my-active'
    });

    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router: routerObj
    });
</script>
</body>
</html>
```



### 11.8 命名视图

有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 `sidebar` (侧导航) 和 `main` (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 `router-view` 没有设置名字，那么默认为 `default`。

```html
<!--这个view默认显示default属性中配置的组件-->
<router-view></router-view>
<!--这个view显示name=left的组件-->
<router-view name='left'></router-view>
<!--这个view显示name=main的组件-->
<router-view name='main'></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 `components` 配置 (带上 s)：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```



如下示例是一个基于命名视图实现的如下图所示的经典布局：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211225214744.png)

**HTML实现**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>路由命名视图</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
    <style>
        body, div, html {
            margin: 0;
            padding: 0;
        }

        .header {
            background-color: lightsalmon;
            height: 80px;
        }

        .container {
            display: flex;
            height: 630px;
        }

        .left {
            width: 300px;
            background-color: cornflowerblue;
        }

        .main {
            width: 1300px;
            background-color: lightpink;
        }
    </style>
</head>
<body>

<div id="app">
    <router-view class="header"></router-view>
    <div class="container">
        <!--这里的name名称是可以自定义的，只需要和路由规则中components属性配置对应即可-->
        <router-view class="left" name="left"></router-view>
        <router-view class="main" name="main"></router-view>
    </div>
</div>


<script type="text/javascript">
    const header = {
        template: '<div>我是header区域</div>'
    }
    const leftBox = {
        template: '<div>我是leftBox区域</div>'
    }
    const rightBox = {
        template: '<div>我是rightBox区域</div>'
    }

    const router = new VueRouter({
        routes: [
            {
                path: '/',
                components: {
                    'default': header,
                    'left': leftBox,
                    'main': rightBox
                }
            }
        ]
    })

    const vm = new Vue({
        el: '#app',
        data: {},
        router: router
    })

</script>
</body>
</html>
```



## 十二、Vue 监听属性

Vue 监听属性 `watch` 可以监控一个值的变换，并调用因为变化需要执行的方法。可以通过watch动态改变关联的状态。

watch属性基本语法：

```js
const vm=new Vue({
    ...
    watch:{
        //filed_name 可以时任何可以在vue实例中使用的属性名
        //需要实现一个方法处理监听到属性变化时的处理逻辑
        //方法可以有两个入参：newVal 和 oldVal ，分别表示属性变化前后的值，由vue负责在调用方法的时候传入
        'filed_name': function (newVal,oldVal){
            //监听到逻辑时的处理逻辑
        }
    }
    ...
})
```



一般watch多用于监听一些vue对象的变化，比如监听vue的路由变化，如下是一个示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vue监听属性实现监听路由变化</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- vue-router源文件 -->
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
</head>
<body>

<div id="app">

    <router-link to="/login">登录</router-link>
    <router-link to="/register">注册</router-link>
    <router-view></router-view>
</div>


<script type="text/javascript">

    const login = {
        template: '<div><h1>登录页面</h1></div>'
    }

    const register = {
        template: '<div><h1>注册页面</h1></div>'
    }

    const router = new VueRouter({
        routes: [
            {
                path: "/",
                redirect: '/login'
            },
            {
                path: "/login",
                component: login
            },
            {
                path: "/register",
                component: register
            }
        ]
    })

    const vm = new Vue({
        el: '#app',
        data: {},
        methods: {},
        router: router,
        watch: {
            //监听路由路径的变化，并给予不同提示
            //需要提供一个方法处理当监听到变化时如何处理该变化，方法可以传入两个参数，newVal 和我 oldVal
            // newVal: 变化前的值  oldVal: 变化后的值
            '$route.path': function (newVal, oldVal) {
                var msg = '访问了'+newVal;
                if (newVal === '/login') {
                    msg = '欢迎来到登录页面！';
                } else if (newVal === '/register') {
                    msg = '欢迎来到注册页面！'
                }
                alert(msg)
            }
        }
    })

</script>

</body>
</html>
```





## 十三、Vue 计算属性

Vue 计算属性 computed 可以用来监控自己定义的变量，该变量不在data里面声明，直接在computed里面定义，然后就可以在页面上进行双向数据绑定展示出结果或者用作其他处理；

computed基本语法：

```js
const vm=new Vue({
    ...
    computed:{
        //filed_name 可以时任何可以在vue实例中使用的属性名
        //需要实现一个方法处理监听到属性变化时的处理逻辑
        'filed_name': function (){
            //监听到逻辑时的处理逻辑...
            //最后computed的方法一定要有return返回值
            return ...
        }
    }
    ...
})
```

computed比较适合对多个变量或者对象进行处理后返回一个结果值，也就是数多个变量中的某一个值发生了变化则我们监控的这个值也就会发生变化，如下是一个根据用户输入姓和名的进而形成完整姓名的案例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>计算属性用法</title>
    <!-- vue源文件 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div id="app">

    姓：<input type="text" v-model:value="firstname">
    名：<input type="text" v-model:value="lastname">

    姓名：<input type="text" v-model:value="fullname">
</div>


<script type="text/javascript">
    const vm = new Vue({
        el: '#app',
        data: {
            firstname:'',
            lastname: ''
        },
        computed: {
            //这里的fullname 时 firstname 和 lastname 拼接而成的，涉及属性计算，类似这种属性可以不定义在data
            'fullname': function () {
                return this.firstname + this.lastname;
            }
        }
    })

</script>

</body>
</html>
```



## 十四、Webpack

Webpack 是一个基于Node.js开发的**前端资源加载/打包工具**。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。

![](http://image.easyblog.top/32af52ff9594b121517ecdd932644da4.png)

从图中我们可以看出，Webpack 可以将多种静态资源 js、css、less 转换成一个静态文件，减少了页面的请求。

### 14.1 安装webpack

在安装 Webpack 前，你本地环境需要支持 [node.js](https://www.runoob.com/nodejs/nodejs-install-setup.html)。

由于 npm 安装速度慢，本教程使用了淘宝的镜像及其命令 cnpm，安装使用介绍参照：[使用淘宝 NPM 镜像](https://www.runoob.com/nodejs/nodejs-npm.html)。



**(1) 使用 cnpm 安装 webpack**

```shell
cnpm install webpack -g
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226155119.png)

**(2) 安装 webpack-cli**

```shell
npm i -g webpack-cli
```

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226210118.png)

### 14.2 webpack 基本使用方式

#### (1) 新建文件夹 `webpack`

新建目录 `webpack-study` ，并在目录下新建 `src` 目录，最后开发所需的 `js` 、`css` 、`images` 以及 `HTML ` 页面等程序源文件都放在这个目录下，并且新建目录 `dist` 用于存放程序编译输出最终可以交付的产品。

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226203255.png)

#### (2) 初始化项目

使用 `npm init -y` 命令初始化项目目录，命令执行成功之后会在项目 根目录下生成 `package.json` 配置文件

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226203155.png)

#### (3) 编写 `main.js`并使用webpack打包 

使用webpack之后就不建议再在项目页面中引入其他依赖的js文件了，这些依赖可以通过自定义的 `main.js` 中引入，在项目中只需要引入自定义的 `main.js` 即可。

```js
//项目的入口js文件

// import xxx from xxx 是ES6导入模块的方式
import $ from 'jquery'


$(function () {
    $('li:odd').css('backgroundColor', 'blue');
    $('li:even').css('backgroundColor', 'yellow')
})
```



之后在项目HTML中直接引入此 `main.js` 即可：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226204159.png)



此时还不能直接打开文件浏览到效果，因为这里我们在 `main.js` 中使用的ES6语法，浏览器无法直接运行，需要使用 webpack 打包成浏览器可以直接运行的低级别语法。在命令行执行命令：`webpack  ./src/main.js -o  ./dist/bundle.js` 

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226212041.png)



虽然执行成功了，但是下边的 `WARNING` 提示看着确实不爽，这个怎么解决呢？

其实看[官网的文档](https://webpack.js.org/configuration/mode/)就知道了，这是webpack让提供环，以便提供不同的优化，more选择的是生成环境模式，这种模式下回将生成的我呢间压缩成一行，便于网络传输，但是不便于开发；因此如果是开发环境，则需要指定 `mode=development`，有两种方式可以指定，一种是直接在命令上使用 `--mode=development` ；第二种是在项目根目录下新建 `webpack.config.js` 配置文件。这里选择第二种方式：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226213545.png)



之后再次执行命令`webpack  ./src/main.js -o  ./dist/bundle.js` ，如下图所示,没有 `WARNING` 提示了，这下看起来就清爽很多....

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226213706.png)

打包成功后就会在项目的 `dist` 目录下生成 `bundle.js` 文件

**development模式下打包生成的文件**

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226213931.png)

**production模式下打包生成的文件**

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226212441.png)



#### (4) 在HTML页面文件中引入 `main.js`

在页面映入经过webpack “翻译'' 过的 `main.js` 到HTML中：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
    <script src="../dist/main.js"></script>
</head>
<body>
<div>
    <ul>
        <li>这是第1个li元素</li>
        <li>这是第2个li元素</li>
        <li>这是第3个li元素</li>
        <li>这是第4个li元素</li>
        <li>这是第5个li元素</li>
        <li>这是第6个li元素</li>
        <li>这是第7个li元素</li>
        <li>这是第8个li元素</li>
        <li>这是第9个li元素</li>
        <li>这是第10个li元素</li>
    </ul>
</div>
</body>
</html>
```

浏览页面，效果被应用上了：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226214857.png)

#### (5) 配置文件源路径和文件输出路径

在第（3）步中我们学会了使用 `webpack 源文件 -o 输出文件` 来打包文件，这种输入命令的方式在日常开发的时候过于麻烦，其实我们也可以将这个信息配置到 `webpack.config.js`  中：

```js
const path = require('path')

module.exports = {
    mode: 'development',
    entry: {
        //配置打包的输入文件路径
        path: path.join(__dirname, './src/main.js')
    },
    output: {
        //配置打包的输出文件路径
        path: path.join(__dirname,'./dist'),
        //指定输出文件名
        filename: 'bundle.js'
    }

}
```

配置完成后，我们现在就可以直接使用 `webpack` 这一个命令来执行打包：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226220213.png)



#### (6) 使用 webpack-dev-server 实现自动打包编译

首先在命令行输入命令 `cnpm i webpack-dev-server` 下载工具

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226221607.png)



在 `package.json` 文件中配置 `scripts`：

```js
"scripts": {
    //配置一个dev环境启动
   "dev": "webpack-dev-server --static ./"
}
```



随后在shell命令行输入命令：`npm run dev` 启动webpack-dev-server，启动后webpack-dev-server会将整个项目默认在8080端口以服务器的形式发布：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226224153.png)



访问 `http://localhost:8080` 可以看到项目的目录结构：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226224240.png)

`index.html` 文件在`src`根目录下，那么访问`src`就可以看到项目首页：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226224404.png)

现在我们可以测试一下修改 `src/main.js` 然后保存文件看看会不会自动打包，如下时效果截图：

![动画](http://image.easyblog.top/%E5%8A%A8%E7%94%BB.gif)

可以看到 `webpack-dev-server` 在保存文件时确实会自动将有修改的文件重新打包了。

虽然可以自动打包了，可是此时刷新浏览器会返现修改并未应用到页面，难道时到不没有成功？其实这是webpack-dev-server的一个优化的地方，它把打包后的bundle放到了内存中，而我们页面引用的还是之前使用webpack打包存放在磁盘上的`bundle.js`。因此在开发阶段，我们可以直接引入内存中这个 `bundle.js`，它在 `/bundle.js`

<img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211226230840.png" style="width:70%;" />

此时，我们修改文件，同时观察页面的变化：

![](http://image.easyblog.top/11.gif)

观察发现，源文件和页面的修改几乎是同步的。



### 14.3 `webpack-dev-server` 常用参数

上面了解了webpack的基本使用方式以及`webpack-dev-server` 配置自动打包的方式。接下来，我们再来了解一下这个开发中常用的工具的一些参数选项：

* `--static src`   自定托管的文件夹路径，默认是当前项目根目录下的`public` 目录
* `--open`   配置打包后自动打开浏览器
* `--port 80`  自定webpack-dev-server启动的端口，默认是8080
* `--hot`   配置热更新，高版本是默认打开的

 

### 14.4 loader

Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 进行转换。比如如果我们需要在应用中添加 css 文件，就需要使用到 `css-loader` 和 `style-loader`，他们做两件不同的事情，`css-loader` 会遍历 CSS 文件，然后找到 url() 表达式然后处理他们，`style-loader` 会把原来的 CSS 代码插入页面中的一个 style 标签中。

接下来我们使用以下命令来安装 css-loader 和 style-loader(全局安装需要参数 -g)。

```shell
cnpm install css-loader style-loader -D
```

执行以上命令后，会在当前目录生成的 node_modules 目录，它就是 css-loader 和 style-loader 的安装目录。

安装完成之后，我们需要在 `webpack.config.js` 配置文件中配置 `.css` 文件的处理规则，让webpack可以借助loader处理 `.css` 文件：

```js
module.exports = {
    ...
    module: {
        //新增第三方文件解析规则
        //test 手机正则表达式 ，表示匹配.css结尾的文件
        //use 表示匹配上的文件使用 css-loader 和 style-loader 依次处理
        rules: [
            {test: /\.css$/, use: ['style-loader', 'css-loader']}
        ]
    }
}
```

在 `main.js` 中使用 `import` 导入 css 文件，并重启webpack-dev-server

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20211227212140.png)



这里只是演示了如何使用webpack打包css文件，其他类型的文件的打包可以参考 [webpack官网-loaders](https://www.webpackjs.com/loaders/)

