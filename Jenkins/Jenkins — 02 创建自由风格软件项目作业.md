## Jenkins — 02 创建自由风格软件项目作业



接[上一节](./Jenkins — 01 Jenkins快速入门) 教程继续，本文以我托管在gitee上的一个开源项目**[spring-boot-ci-test](https://gitee.com/LoveITer/spring-boot-ci-test)**做为演示，讲解如何创建自动化部署。





## 一、创建项目

在Jenkins后台点击 `【新建任务】-> 【构建一个自由风格的软件项目】` 跳转到如下页面输入任务名称，点击下方确定按钮开始配置任务构建的一些详细信息。

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%882.28.41.png)

首先是一些通用的参数配置 `【General】`，可以根据自己项目进行配置

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%882.47.36.png)



然后是 `【源代码管理】` 部分，这是一个git项目，先在Repository URL这里填写上项目的git地址（即：红字1的位置），git是需要用户名密码才能访问的，所以Credentials这里要选择相应的用户名、密码(即：红字2的部分），红字3的部分为git获取的源代码分支名称，一般为master主分支，也可以改成自己希望的分支。

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%883.16.15.png)



第三步，配置自动化构建触发器，Jenkins提供了多种触发方式，这里选择使用同步URL+Token 令牌验证的方式来进行触发。调用这个URL就可以触发Jenkins自动构建并部署项目到自定的服务器上去。下图所示是钩子URL配置示例：

![](http://image.easyblog.top/%E6%88%AA%E5%B1%8F2021-08-25%20%E4%B8%8B%E5%8D%883.11.11.png)

有了钩子触发URL，还需要一个触发钩子的操作，本教程使用的是gitee，在仓库管理中找到 `WebHook` 配置上面的URL钩子到git仓库中，之后在下面设置那种情况下触发自动构建。

