---
title: VUE笔记
date: 2021-11-09 12:07:09
tags: 学习
---

# Vue

SOC关注点分离原则

视图层：给用户看，刷新后台给的数据

网络通信：axios

页面跳转：vue-router

状态管理：vuex

vue-ui：ICE

webpack：打包工具类似于maven

Angular：前端MVC模式

React：虚拟DOM：利用内存；

vm：数据双向绑定

计算属性->Vue特色

MVVM+虚拟DOM=VUE是两大框架融合

```vue
<!--view  层模板-->
<div id="app">
    {{message}}
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            message:"hello vue"
        }
    });
</script>
view modle
点开页面，console->vm.massage="随意字符"
负责转化modle层的数据对象让数据变得更容易管理和使用可以与视图层双向绑定
这被称为虚拟DOM因为没有操作DOM
// 鼠标悬停方法
<span v-bind:title="message">
        鼠标悬停
</span>
```

HTML+CSS+Templates VIEW视图层

​     |       双向绑定            |

JAVASCRIPT Runtime（及时运行） Compiler（及时编译） VIEW MODEL

​     |     AJAX                    |     JSON

model

​     |

JAVA

​     |

数据库

VUE:else-if语法

```vue
<!--view  层模板-->
<div id="app">
    <h1 v-if="type==='A'">A</h1>
    <h1 v-else-if="type==='B'">B</h1>
    <h1 v-else-if="type==='D'">D</h1>
    <h1 v-else>C</h1>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            type:"B"
        }
    });
</script>
```

VUE:for语法

```vue
<!--view  层模板-->
<div id="app">
    <h1 v-for="item in items">
        {{item.message}}
    </h1>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            items:[
                {message:"蒙"},
                {message:"园"},
                {message:"青"},
                {message:"牛"}
            ]
        }
    });
</script>
```

## 事件

```vue
<!--view  层模板-->
<div id="app">
    <button v-on:click="sayHi">
        click me
    </button>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            message:"园青学VUE"
        },
        methods:{
            <!--方法必须定义在vue的methods中v-on事件,-->
            sayHi:function (event) {
                alert(this.message);
            }
        }
    });
</script>
```

双向绑定

```vue
<!--view  层模板-->
<div id="app">
    输入的文本：<input type="text" v-model="message">{{message}}
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            message:"123"
        }
    });
</script>
<input> <textarea> <select> 这三个标签可以实现双向绑定
```

双向绑定之下拉框

```vue
<!--view  层模板-->
<div id="app">
    <select v-model="selected">
        <option value="" disabled>请选择</option>
        <option>A</option>
        <option>B</option>
        <option>C</option>
    </select>
    <span>value:{{selected}}</span>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            selected:''
        }
    });
</script>
```

## 组件

小坑：Vue写成了vue

先加载模板再渲染数据

```vue
<!--view  层模板-->
<div id="app">
    <yuanqing v-for="item in items" v-bind:yuan="item">

    </yuanqing>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    // 定义一个VUE组件
    Vue.component("yuanqing",{
        props:['yuan'],
        template:'<li>{{yuan}}</li>'
    });
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            items:["APP","backend","web"]
        }
    });
</script>
```

axios

```vue
<!--view  层模板-->
<div id="vue" v-cloak>
    <div>{{info.name}}</div>
    <div>
        {{info.address.country}}
    </div>
    <a v-bind:href="info.url">来点我</a>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script>
    var vm = new Vue({
        el:"#vue",
        // 这个是函数
        data(){
            return{
                // 请求的返回参数格式，必须和json字符串一样
                info:{
                    name:null,
                    address:{
                        street:null,
                        city:null,
                        country:null,
                    },
                    url:null,
                }
            }
        },
        mounted(){// 钩子函数,vue的声明周期中，mounted最适合操作json
          	// response.data这个data是属性
            axios.get('data.json').then(response=>(this.info=response.data))
        }
    });
</script>
```

![img](/typora-user-images/2020032311230745.png)

计算属性：计算出来的结果保存在属性中，内存中运行：虚拟DOM，可以想象成一个缓存

```vue
<!--view  层模板-->
<div id="app">
    <p>currentTime1{{currentTime1()}}</p>
    <p>currentTime2{{currentTime2}}</p>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    var vm = new Vue({
        el:"#app",
        // model: 数据
        data:{
            message:'hello，yuanqing'
        },
        methods:{
            currentTime1:function () {
                return Date.now();// 返回一个时间戳
            }
        },
        computed:{// 计算属性：methods，computed方法名不建议重名，重名之后只会调用methods中的方法
            currentTime2:function () {
                return Date.now();// 返回一个时间戳
            }
        }
    });
</script>
```

computed (计算属性) 和 methods 的区别？

当依赖的属性发生改变时，计算属性与方法都会被重新调用，此时二者没有区别。 

当依赖的属性再次调用时，计算属性会自动获取数据的缓存，而不是重新调用计算属性计算过程。 

而无论依赖的属性方法是否发生变化，只要再次调用方法，就会重新执行方法的内容。 所以计算属性的效率更高



组件插槽 slot

```vue
<!--view  层模板-->
<div id="app">
    <p>列表数据</p>
    <ul>
        <li>java</li>
        <li>java</li>
        <li>java</li>
    </ul>
    <todo>
        <todo-title slot="todo-title" :title="title"></todo-title>
        <todo-items slot="todo-items" v-for="item in todoItems" :item="item"></todo-items>
    </todo>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    Vue.component("todo",{
        template: '<div>\
                  <slot name="todo-title"></slot>\
                    <ul>\
                        <slot name="todo-items"></slot>\
                    </ul>\
                  </div>'
    });
    Vue.component("todo-title",{
        props:['title'],
        template: '<div>{{title}}</div>'
    })
    Vue.component("todo-items",{
        props: ['item'],
        template:'<li>{{item}}</li>'
    })

    var vm = new Vue({
        el:"#app",
        data:{
            title:"精日少年刘伯文",
            todoItems:['A','B','C']
        }
    });
</script>
```

事件

```vue
<!--view  层模板-->
<div id="app">
    <todo>
        <todo-title slot="todo-title" :title="title"></todo-title>
        <todo-items slot="todo-items" v-for="(item,index) in todoItems" :item="item" :index="index" v-on:remove="removeItems(index)" key="index"></todo-items>
    </todo>
</div>
<!--导入vue.js-->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.min.js"></script>
<script>
    Vue.component("todo",{
        template: '<div>\
                  <slot name="todo-title"></slot>\
                    <ul>\
                        <slot name="todo-items"></slot>\
                    </ul>\
                  </div>'
    });
    Vue.component("todo-title",{
        props:['title'],
        template: '<div>{{title}}</div>'
    })
    Vue.component("todo-items",{
        props: ['item','index'],
        // 只能绑定当前组件的方法
        template:'<li>{{index}}-------{{item}} <button @click="remove">删除</button></li>',
        methods: {
            remove:function (index) {
                // 自定义事件分发
                this.$emit('remove',index)
            }
        }
    })

    var vm = new Vue({
        el:"#app",
        data:{
            title:"精日少年刘伯文",
            todoItems:['A','B','C']
        },
        methods: {
            removeItems:function (index) {
                console.log("删除了"+this.todoItems[index]+"OK")
                this.todoItems.splice(index,1);// 一次删除一个元素
            }
        }
    });
</script>
```

![image-20210223123129758](/typora-user-images/image-20210223123129758.png)

## vue-cli

 vue-cli 官方提供的一个脚手架,用于快速生成一个 vue 的项目模板;
 预先定义好的目录结构及基础代码，就好比咱们在创建 Maven 项目时可以选择创建一个骨架项目，这个骨架项目就是脚手架,我们的开发更加的快速;
主要的功能:

* 统一的目录结构
* 本地调试
*  热部署
*  单元测试
*  集成打包上线

需要的环境
Node.js : http://nodejs.cn/download/
安装就无脑下一步就好,安装在自己的环境目录下
Git : https://git-scm.com/downloads
镜像:https://npm.taobao.org/mirrors/git-for-windows/
确认nodejs安装成功:
cmd 下输入 node -v,查看是否能够正确打印出版本号即可!
cmd 下输入 npm-v,查看是否能够正确打印出版本号即可!
这个npm,就是一个软件包管理工具,就和linux下的apt软件安装差不多!
安装 Node.js 淘宝镜像加速器（cnpm）
这样子的话,下载会快很多~
在命令台输入

* 此处有一小坑如果是苹果系统需要在命令前加sudo，否则无法获得管理员权限

```shell
# -g 就是全局安装
npm install cnpm -g
# 或使用如下语句解决 npm 速度慢的问题
npm install --registry=https://registry.npm.taobao.org
```

安装的位置:C:\Users\Administrator\AppData\Roaming\npm

### 安装vue-cli

* 此处有一小坑如果是苹果系统需要在命令前加sudo，否则无法获得管理员权限

```shell
#在命令台输入
cnpm install vue-cli -g
#查看是否安装成功
vue list
```

创建一个基于webpack模板的vue程序

```shell
# 这里的 myvue 是项目名称，可以根据自己的需求起名
vue init webpack myvue
```

一路都选择no即可;
初始化并运行

```shell
cd myvue
npm install
npm run dev
```

依赖存放在工程中node_modules下

## Webpack

 WebPack 是一款**模块加载器兼打包工具**，它能把各种资源，如 JS、JSX、ES6、SASS、LESS、图片等**都作为模块来处理和使用。**

```shell
npm install webpack -g # 报错带sudo
npm install webpack-cli -g
```

测试安装成功: 输入以下命令有版本号输出即为安装成功

```shell
webpack -v
webpack-cli -v
```

## 什么是Webpack

 本质上，webpack是一个现代JavaScript应用程序的静态模块打包器(module bundler)。当webpack处理应用程序时，它会递归地构建一个依赖关系图(dependency graph),其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个bundle.
 Webpack是当下最热门的前端资源模块化管理和打包工具，它可以将许多松散耦合的模块按照依赖和规则打包成符合生产环境部署的前端资源。还可以将按需加载的模块进行代码分离，等到实际需要时再异步加载。通过loader转换，任何形式的资源都可以当做模块，比如CommonsJS、AMD、ES6、 CSS、JSON、CoffeeScript、LESS等;
 伴随着移动互联网的大潮，当今越来越多的网站已经从网页模式进化到了WebApp模式。它们运行在现代浏览器里，使用HTML5、CSS3、ES6 等新的技术来开发丰富的功能，网页已经不仅仅是完成浏览器的基本需求; WebApp通常是一个SPA (单页面应用) ，每一个视图通过异步的方式加载，这导致页面初始化和使用过程中会加载越来越多的JS代码，这给前端的开发流程和资源组织带来了巨大挑战。
 前端开发和其他开发工作的主要区别，首先是前端基于多语言、多层次的编码和组织工作，其次前端产品的交付是基于浏览器的，这些资源是通过增量加载的方式运行到浏览器端，如何在开发环境组织好这些碎片化的代码和资源，并且保证他们在浏览器端快速、优雅的加载和更新，就需要一个模块化系统，这个理想中的模块化系统是前端工程师多年来一直探索的难题。

## webpack demo

1.先创建一个包 交由idea打开 会生成一个.idea文件 那么就说明该文件就交由idea负责

2.在idea中创建modules包，再创建hello.js
hello.js 暴露接口 相当于Java中的类

```js
exports.sayHi = function () {
    document.write("<h1>hello world</h1>")
}
```

3.创建main.js 当作是js主入口
main.js 请求hello.js 调用sayHi()方法

```js
let hello=require("./hello");
hello.sayHi()
```

4.在主目录创建webpack-config.js
webpack-config.js 这个相当于webpack的配置文件 enrty请求main.js的文件 output是输出的位置和名字

```js
module.exports = {
    entry: './modules/main.js',
    output: {
        filename: "./js/bundle.js"
    }
};
```

5.在idea命令台输入webpack命令（idea要设置管理员启动）

6.完成上述操作之后会在主目录生成一个dist文件 生成的js文件夹路径为/dist/js/bundle.js

7.在主目录创建index.html 导入bundle.js
index.html

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="dist/js/bundle.js"></script>
</head>
<body>
</body>
</html>
```

路由流程讲解

```vue
App.vue是首页，通过<router-link to="/main"></router-link>这个标签中的router
找到main.js的 由于你在main.js中配置了router 他就会找到router
通过router里跳转的组件会跳到对应页面
```

1.新建Main Content页面

2.安装路由,在src目录下,新建一个文件夹 : router,专门存放路由

index.js(默认配置文件都是这个名字)

```js
import Vue from 'vue'
import Router from 'vue-router'

import Content from "../components/Content";
import Main from "../components/Main";
import Yuan from "../components/Yuan";

// 安装路由
Vue.use(Router);

// 配置导出路由
export default new Router({
  routes:[
    {
      // 路由路径
      path:'/content',
      name:'content',
      // 跳转的组件
      component:Content
    },
    {
      // 路由路径
      path:'/main',
      name:'main',
      // 跳转的组件
      component:Main
    },
    {
      // 路由路径
      path:'/yuan',
      name:'yuan',
      // 跳转的组件
      component:Yuan
    }
  ]
});
```

3.在main.js中配置路由

main.js

```js
import Vue from 'vue'
import App from './App'
import router from './router' // 自动扫描里面的路由配置

Vue.config.productionTip = false


new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})
```

4.在App.vue中使用路由

App.vue

```vue
<template>
  <div id="app">
    <h1>vue-router</h1>
    <router-link to="/main">首页</router-link>
    <router-link to="/content">内容页</router-link>
    <router-link to="./yuan">园青专属</router-link>
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

vue+elementUI实战
根据之前创建vue-cli项目一样再来一遍 创建项目
1． 创建一个名为 hello-vue 的工程 vue init webpack hello-vue
2． 安装依赖，我们需要安装 vue-router、element-ui、sass-loader 和 node-sass 四个插件
