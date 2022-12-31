### Vue实战— 在vue项目中使用element-ui

> 本文主要记录了在使用vue+element ui开发easyblog前端页面时的一些技术关键



创建vue请参考我的另一篇文章 [Vue实战—手把手教你用vue-cli搭建vue项目](./Vue实战—手把手教你用vue-cli搭建vue项目)，本文从创建项目后的依赖安装开始记录。



### 一、安装并配置element-ui插件

可以在vue-cli图形化界面安装依赖：【安装插件】->【搜索“vue-cli-plugin-element”】->【选择“vue-cli-plugin-element”】->【点击安装vue-cli-plugin-element按钮】；或者也可以使用命令安装：`npm install element-ui -S`

![](C:/Users/HuangXin/Pictures/QQ%E6%88%AA%E5%9B%BE20220226144132.png)

安装成功之后需要配置插件，需要配置`element-ui` 按需导入：

![](http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220226144502.png)



### 二、在项目中使用ElementUI

**（1）在src/plugins/element.js目录中导入项目所需组件**

`element.js` 文件主要配置项目中使用到的 element-ui 组件，这里建议按需导入，尽可能减少最终输出`dist`包的体积。在 `element.js` 文件中做如下配置：

```javascript
import Vue from 'vue'
import {
  Button //按需导入组件
} from 'element-ui'

//使用组件
Vue.use(Button)

//挂载 element-ui 的message 组件到 vue 实例
//Vue.prototype.$message = Message
//Vue.prototype.$confirm = MessageBox.confirm
```

**（2）在main.js中引导入element.js**

正常清空下安装并配置 `element-ui` 之后就会自动在 `main.js` 中增加对element的引用，如果没有的话，可以参考如下代码导入。

```javascript
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import './plugins/element'  //导入element组件

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```



**（3）在vue源文件中使用element组件**

新建vue文件 `LoginView.vue` 并在文件中编写如下内容：

```vue
<template>
  <div class="login-container">
    <div class="login-box">
      <div class="avatar-box">
        <img src="../assets/logo.png">
      </div>
      <!--登录表单-->
      <el-form  class="login-form">
        <!---用户名-->
        <el-form-item prop="=username">
          <el-input placeholder="用户名" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>

        <!--密码-->
        <el-form-item prop="password">
          <el-input placeholder="密码"  prefix-icon="iconfont icon-3702mima" type="password"
                    show-password></el-input>
        </el-form-item>

        <el-form-item class="btns">
          <el-button plain type="primary">登录</el-button>
          <el-button plain type="info">注册</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LoginView'
}
</script>

<style scoped>
.login-container {
  background-color: #2b4b6b;
  height: 100%;
}

.login-box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}

.avatar-box {
  height: 130px;
  width: 130px;
  border: 1px solid #eee;
  border-radius: 50%;
  padding: 10px;
  box-shadow: 0 0 10px #ddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -40%);
  background-color: #fff;
}

.avatar-box img {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background-color: #eee;
}

.btns {
  display: flex;
  justify-content: flex-end;
}

.login-form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
</style>
```



实现效果如下：

<img src="http://image.easyblog.top/QQ%E6%88%AA%E5%9B%BE20220226154034.png" style="width:90%;" />

