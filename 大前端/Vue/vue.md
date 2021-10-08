



* 1.Vue基础
* 2.Vue常用指令
    * 2.1 vue的使用3部曲
    * 2.2 插值表达式
    * 2.3 文本指令：v-text 和 v-html
    * 2.4 事件指令v-on、控制指令v-model（双向绑定）、按键修饰符 和 事件修饰符
    * 2.5 条件指令v-if、显示指令v-show
    * 2.6 属性指令v-bind
    * 2.7 循环指令v-for
* 3.vue中的组件
    * 3.1 全局组件(父子组件通信)
    * 3.2 局部组件
    * 3.3 插槽
* 4.Vuex
    * 需求：视图将数据放入store全局数据中，并从store中获取全局数据进行展示
    * ​4.1 自定义登录Login组件 
    * 4.2 将数据存储到store中
    * 4.3 view视图使用store中数据
* 5.vue-cli脚手架
    * 5.1 环境准备
    * 5.2 安装脚手架
    * 5.3 第一个vue脚手架项目
        * 安装axios
        * 安装element-ui
    * 5.4 vue-cli脚手架项目打包和部署



## 1.Vue基础

```markdown
# npm项目下载js资源
	npm init -y          -- 初始化npm
	npm install vue      -- 下载vue的js文件
	npm install axios    -- 下载axios的js文件
	npm install jquery   -- 下载jquery的js文件

# vue小结
	Vue 是一个javascript 框架

# Vue的优势
	- 将数据和试图分离，通过改变数据来达到改变试图的效果，便于后期维护
	- 所有的指令和axios都是围绕data进行展开
	
	注：mvc m：model数据模型（数据库）  v：view视图（jsp）  c：controller控制层
	
# vue生命周期
	- 1.new Vue()
	- 2.beforeCreate
	- 3.created：可以拿到data。 **实际用的多**
	- 4.beforeMount
	- 5.mounted  **实际用的多**
  		- beforeUpdate（数据更改才触发）
  		- updated（数据更改才触发）
	- 6.beforeDestroy
	- 7.destroyed（销毁vue。实际不会这么做）
	
```




## 2. Vue常用指令

- 文本指令：v-text 和 v-html
- 事件指令：v-on。可以简写，例如：@click 是 v-on:click的缩写
- 属性指令：v-bind。可以简写，例如：:class 是v-bind:class 的缩写
- 控制指令：v-model。管理form元素，双向绑定数据
- 循环指令：v-for
- 条件指令：v-if
- 显示指令：v-show （因为有v-if的存在所以不怎么常用v-show）


### 2.1 vue的使用3部曲


```markdown
  vue使用3部曲
    1.导入vue.js
    2.创建一个vue对象  表示从app节点开始渲染节点内的节点
    3.定义vue视图
```



```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>

<body>

    <!--3.定义vue视图-->
    <div id="app">
    
    <!--1.导入vue.js-->
    <script src="../js/vue.min.js"></script>

    <script>
        // 2.创建一个vue对象  表示从app节点开始渲染节点内的节点
        // 定义vue的model模型
        var vm = new Vue({
            el:"#app",
            data:{},
            methods:{},
            // 生命周期钩子
            created(){},
            // 组件属性
            props:{},
            // 计算属性（相比methods具有监听作用）
            comments:{},
            // 监听属性
            watch:{},
            // 局部组件
            components:{}
        });
    </script>

</body>
</html>
```



### 2.2 插值表达式

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>

    <!--插值表达式-->
    <div id="app">
        我的名字：{{title}}
        <hr>
        <p>我的性别：{{male==0?"男":"女"}}</p>
        <hr>
        <p>我的年龄：{{myfilter(age,10)}}</p>
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        var vm = new Vue({
            el:"#app",
            data:{
                title:"我是君莫笑",
                male:1,
                age:20
            }.
            methods:{
                myfilter:function (val,num) {
                    return val + num;
            }
        });
    </script>

</body>
</html>
```


### 2.3 文本指令：v-text 和 v-html

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>
    
    <!--v-text 和 v-html（文本指令） ：和插值表达式都具有计算、三目、调用内置方法-->
    <div id="app">
        <p v-html="content"></p>
        <p v-text="male==0?'男':'女'"></p>
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        var vm = new Vue({
            el:"#app",
            data:{
                male:1,
                content:"<strong>我是带标签的文本</strong>"
            }
        });
    </script>
</body>
</html>
```


### 2.4 事件指令v-on、控制指令v-model（双向绑定）、按键修饰符 和 事件修饰符

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>
    
    <!--1.事件指令v-on-->
    <!--2.控制指令v-model-->
    <!--3.按键修饰符 和 事件修饰符-->
    
    <div id="app">   
        <!--1.1 单击事件  @click 是 v-on:click的缩写-->
        <!--注：按键修饰符：回车后直接触发提交方法  @keyup.enter-->
        <p>用户：<input type="text" v-model="user.userName"></p>
        <p>密码：<input type="text" v-model="user.password"></p>
        <!--
            <p>密码：<input type="text" v-model="user.password" @keyup.enter="login"></p>
        -->
        <p><button @click="login">登录</button></p>


        <!--注：事件修饰符：抑制默认行为的发生  例如： @click.prevent中的prevent-->
        <!--默认行为：a button input submit ...-->
        <a href="https://www.baidu.com" @click.prevent="gotoBaidu">点击我触发跳转事件</a>
        <!--
            <a href="#" @click.prevent="gotoBaidu">点击我触发跳转事件</a>
            <a href="javascript:void(0);" @click ="gotoBaidu">点击我触发跳转事件</a>
        -->  
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        var vm = new Vue({
            el:"#app",
            data:{
                user:{}
            },
            methods:{
                login:function () {
                    console.log("你点击了我");
                    var userName = this.user.userName;
                    var password = this.user.password;
                    alert("我的名字和密码是：" + userName + password)
                },
                gotoBaidu:function (obj) {
                    alert("去百度..." + obj.name);
                    return "i love you";
                }
            }
        });
    </script>
</body>
</html>
```


### 2.5 条件指令v-if、显示指令v-show

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>
    <!--v-if 和 v-show 显示或者隐藏-->
    <div id="app">
        <!--v-if-->
        <p v-if="male==1">我的性别：男</p>
        <!--v-show-->
        <p v-show="flag">我是一个p标签</p>
        <button @click="hideFlag">隐藏</button>
        <button @click="showFlag">显示</button>
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        var vm = new Vue({
            el:"#app",
            data:{
                male:1,
                flag:true
            },
            methods:{
                hideFlag:function () {
                    this.flag = false
                },
                showFlag:function () {
                    this.flag = true
                }
            }
        });
    </script>
</body>
</html>
```


### 2.6 属性指令v-bind

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>
 	<!--属性指令（管理属性）  将属性与data绑定  :class 是v-bind:class 的缩写 -->
    <div id="app">
        <span :content="content"></span>      
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        var vm = new Vue({
            el:"#app",
            data:{
                content:"<strong>我是带标签的文本</strong>"
            }
        });
    </script>
</body>
</html>
```



### 2.7 循环指令v-for

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>
	<!--v-for-->
    <div id="app">
        <p v-for="(user,index) in users">
            {{index + 1}} , {{user.name}} , {{user.age}}
        </p>
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        var vm = new Vue({
            el:"#app",
            data:{
                users:[
                    {
                        name:"zhangsan",
                        age:"14"
                    },
                    {
                        name:"xiaoming",
                        age:"12"
                    },
                    {
                        name:"xiaohong",
                        age:"13"
                    }
                ]
            }
        });
    </script>
</body>
</html>
```


## 3.vue中的组件

### 3.1 全局组件(父子组件通信)

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>

    <!--全局组件-->
    <div id="app">
        <!--全局组件的使用-->
        <yl-button youli="我是youli按钮"></yl-button>
        <!--1.自定义属性：父组件数据传给子组件-->
        <!--子组件绑定父组件 用父组件给子组件传值 v-bind-->
        <yl-button :youli="title"></yl-button>
        <!--2.自定义事件：子组件将数据传给父组件-->
        <!--子组件触发父组件事件  this.$emit-->
        <yl-button @myevent="gotoBaidu">触发父组件事件</yl-button>
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        // 全局组件：vue组件的全局注册
        Vue.component('yl-button',{
            // 如果没有传"youli"属性，就用cname作为默认值
            template:"<button @click='clickme'>{{youli || cname}}</button>",
            // props：自定义属性  解决父组件给子组件传值（通过 v-bind 绑定）
            props: ["youli"],
            data:function () {
                return{
                    cname:"我是一个按钮"
                }
            },
            methods:{
                clickme:function () {
                    alert("我是组件的事件");
                    this.$emit('myevent',{name:"youli"})
                }
            },
            created:function () {
                this.cname = "我是按钮";
            },
            computed:function () {},
            watch:function () {}
        });
        
        var vm = new Vue({
            el:"#app",
            data:{
                title:"我是君莫笑"
            },
            methods:{
                gotoBaidu:function (obj) {
                    // obj.name = "我是君莫笑"
                    alert("去百度..." + obj.name);
                    return "i love you";
                }
            },
        });
    </script>
</body>
</html>
```



### 3.2 局部组件

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>

    <div id="app">
 
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        var vm = new Vue({
            el:"#app",
            data:{
                content:"<strong>我是带标签的文本</strong>"
            },
            // 局部组件
            components:{
                'ylxjq-button':{
                    template: "<button @click='clickme'>{{title}}</button>",
                    data:function () {
                        return{
                            title:"我是一个按钮"
                        }
                    },
                    methods:{
                        clickme:function () {
                            alert("我是组件的事件")
                        }
                    }
                }
            }
        });
    </script>
</body>
</html>
```


### 3.3 插槽

```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>vue史诗级入门教程</title>
</head>
<body>
    <!--插槽的使用  <slot>-->
    <div id="app">  
        <xjq-button>新增</xjq-button>
        <xjq-button>修改</xjq-button>
        <xjq-button>删除</xjq-button>
    </div>

    <script src="../js/vue.min.js"></script>
    <script>
        Vue.component('xjq-button',{
            template:"<button><slot></slot></button>"
        });
        
        var vm = new Vue({
            el:"#app",
            data:{}
        });
    </script>
</body>
</html>
```



## 4.Vuex

```markdown
# Vuex的作用
	- store文件夹的index.js
	- 全局存储。vue页面多组件共享数据。相当于java中的session给所有java的类使用
```



### 需求：视图将数据放入store全局数据中，并从store中获取全局数据进行展示

### 4.1 自定义登录Login组件

```vue
<template>
  <h3>您当前登录用户是：{{user.nickName}}</h3>
</template>
<script>

import { mapState } from 'vuex'
    
export default {
  name: 'Login',
  props:{
      msg: String
  },
  components: {
  },
  // vuex使用解构mapState。因为实际开发中，除了vuex状态需要响应数据，当前页面还需要控制计算属性。
  computed:{
    // 从store中拿到用户数据  等价于"return this.$store.state.user"
    ...mapState({user:'user'})
  }
}
</script>
```



### 4.2 将数据存储到store中


```js
import Vue from 'vue'
// 1.引用vuex
import Vuex from 'vuex'

// 2.注册vuex插件到vue中 这里是全局注册
Vue.use(Vuex)

// 3.开始给vuex中定义响应式全局数据共享对象this.$store
export default new Vuex.Store({
  // 3.1 定义响应式的数据  类似于data，只不过是全局
  state: {
  	user:{
      nickName:"",
      password:""
    }
  },
  // 3.2 定义改变state数据的行为。直接改变state
  // params从"actions"来
  // commit调用
  // 注：相当于生产线
  mutations: {
    changeUser(state,params){
      state.user.nickName = params.nickName;
      state.user.password = params.params;
    } 
  },
  // 3.3 actions是处理异步数据加载和提交mutations的机制。间接改变state
  // params数据从页面来
  // commit注册  dispatch调用
  // 注：相当于车间（注册生产线）
  actions: {
    login({commit},params){
      commit("changeUser",params);
    }
  },
  // 3.4 对状态state管理的一种过滤处理的机制（实际可以不用）
  getters:{},
  // 3.5 模块化隔离的一种机制（实际可以不用）
  modules: {}
})
```


### 4.3 view视图使用store中数据

```vue
<template>
  <div class="home">
    <Login></Login>  // 使用自定义组件包
    <button @click="toLogin">登录</button>
  </div>
</template>
<script>

// 1.引入自定义组件包   mapActions,mapMutations等叫做解构
import Login from '@/components/Login.vue'
import { mapActions } from 'vuex'
import { mapMutations } from 'vuex'
    
export default {
  name: 'Home',
  // 2.注册自定义组件包
  components: {
    Login
  },
  data(){
    return{
        nickName:"游利",
        password:"xjq"
    } 
  },
  created(){},
  computed:{}, // 相比于methods中的方法，这里面的方法具有响应式（监听）的作用。
  methods:{
    // 3.将数据存到store缓存中  等价于"this.$store.dispatch("login",{nickName,password})"
    ...mapActions({login:'login'}),
    // ...mapMutations({changeUser:'changeUser'})
    toLogin(){
        var nickName = "游利";
        var password = "xjq";
        this.login({nickName,password});
        // this.changeUser({nickName,password});
    }
  }
}
</script>
```


## 5.vue-cli脚手架

### 5.1 环境准备

```markdown
# 0.vue-cli脚手架
	可以理解为java中的maven。
	java之前没有maven需要自己引入jar包开发，有了maven只需要引入maven坐标
	
# 1.安装nodejs环境
	node -v
	npm -v 
	
# 2.npm介绍
	node package mangager    nodejs包管理工具 
		maven 管理java后端依赖    远程仓库(中心仓库)      阿里云镜像
		npm   管理前端系统依赖    远程仓库(中心仓库)      配置淘宝镜像
		
# 3.配置淘宝镜像
	npm config set registry https://registry.npm.taobao.org
	npm config get registry
	
# 4.配置npm下载依赖位置
	 windows:
		npm config set cache "D:\nodereps\npm-cache"
		npm config set prefix "D:\nodereps\npm_global"
	 mac os:
	 	npm config set cache "/Users/chenyannan/dev/nodereps"
		npm config set prefix "/Users/chenyannan/dev/nodereps"
		
# 5.验证nodejs环境配置
	npm config ls
	
```



### 5.2 安装脚手架

```markdown
# 0.卸载脚手架
	npm uninstall -g @vue/cli  //卸载3.x版本脚手架
	npm uninstall -g vue-cli   //卸载2.x版本脚手架

# 1.Vue Cli官方网站
	https://cli.vuejs.org/zh/guide/

# 2.安装vue Cli
	// npm install -g vue-cli
	npm install -g @vue/cli-init
	
```


### 5.3 第一个vue脚手架项目

```markdown
# 1.创建vue脚手架第一个项目
	vue init webpack 项目名   （推荐）

	或者 vue ui 图形化界面创建
# 2.创建第一个项目
	hello     ------------->项目名
    -build    ------------->用来使用webpack打包使用build依赖
    -config   ------------->用来做整个项目配置目录
    -node_modules    ------>用来管理项目中使用依赖
    -src		    ------>用来书写vue的源代码[重点]
    	+assets     ------>用来存放静态资源 [重点]
      components     ------>用来书写Vue组件 [重点]
      router	     ------>用来配置项目中路由[重点]
      App.vue      ------>项目中根组件[重点]
      main.js      ------>项目中主入口[重点]
    -static        ------>其它静态
    -.babelrc      ------> 将es6语法转为es5运行
    -.editorconfig ------> 项目编辑配置
    -.gitignore    ------> git版本控制忽略文件
    -.postcssrc.js ------> 源码相关js
    -index.html    ------> 项目主页
    -package.json  ------> 类似与pom.xml 依赖管理  jquery 不建议手动修改
    -package-lock.json ----> 对package.json加锁
    -README.md         ----> 项目说明文件

# 3.如何运行在项目的根目录中执行
		npm start 或者 npm run dev   

# 4.如何访问项目
		http://localhost:8080  

# 5.Vue Cli中项目开发方式
	 注意: 一切皆组件   一个组件中   js代码  html代码  css样式

```


#### 安装axios

```markdown
# 1.安装axios
	npm install axios --save-dev

# 2.配置main.js中引入axios
	import axios from 'axios';   // 引入axios

	Vue.prototype.$http=axios;   // 修改内部$http为axios

# 3.使用axios
	在需要发送异步请求的位置:
		this.$http.get("url").then((res)=>{})
    	this.$http.post("url").then((res)=>{})
```



#### 安装element-ui

```markdown
# 1.vue-cli中引入elements
npm install element-ui -S

vue-cli中引入element-ui：https://www.cnblogs.com/Pecci/p/11803512.html
```



### 5.4 vue-cli脚手架项目打包和部署

```markdown
# 1.在项目根目录中执行如下命令:
	  npm run build  打包

# 2.打包之后当前项目中变化
	 在打包之后项目中出现dist目录,dist目录就是vue脚手架项目的直接部署目录

# 3.运行及部署
	运行方式1：
    	- 1."npm run dev"或者"npm start"启动前端项目
    	- 2.启动springboot项目
    	- 3.访问前端vue项目
	运行方式2：
    	将前端vue项目打包后的静态资源放到springboot项目的static目录下 或者 放到nginx反向代理后端服务
    	- 访问地址：http://127.0.0.1:8989/vue/dist/index.html
    	- 跨域问题：@CrossOrigin
    
# 注：后端程序员可以采用将vue打包后，将dist文件放入spingboot项目后部署 
# 注：bable：将es6转成es5的工具
# 注：webpack：打包工具

```

