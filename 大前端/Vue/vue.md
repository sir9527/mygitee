

* 1.Vue基础
    * 1.1 Vue的优势
    * 1.2 Vue常用指令
    * 1.3 Vue生命周期
* 2.Vuex
    * 2.1 vue脚手架之vue-cli
    * 2.2 使用vuex
        * 2.2.1 vuex全局存储store数据
        * 2.2.2 view视图使用store中数据
        * 2.2.3 自定义组件




## 1.Vue基础

```vue
npm init -y          -- 初始化npm
npm install vue      -- 下载vue的js文件
npm install axios    -- 下载axios的js文件
npm install jquery   -- 下载jquery的js文件
```



### 1.1 Vue的优势

- 将数据和试图分离，通过改变数据来达到改变试图的效果，便于后期维护
- 所有的指令和axios都是围绕data进行展开


![mvc模式图](./image/mvc模式.png)






### 1.2 Vue常用指令

- 文本指令：v-text 和 v-html
- 事件指令：v-on。可以简写，例如：@click 是 v-on:click的缩写
- 属性指令：v-bind。可以简写，例如：:class 是v-bind:class 的缩写
- 控制指令：v-model。管理form元素，双向绑定数据
- 循环指令：v-for
- 条件指令：v-if
- 显示指令：v-show 



### 1.3 Vue生命周期

- 1.new Vue()
- 2.beforeCreate
- 3.created：可以拿到data。**实际用的多**
- 4.beforeMount
- 5.mounted **实际用的多**
  - beforeUpdate（数据更改才触发）
  - updated（数据更改才触发）
- 6.beforeDestroy
- 7.destroyed（销毁vue。实际不会这么做）



## 2.Vuex

Vuex的作用
- store文件夹的index.js
- 全局存储。vue页面多组件共享数据。相当于java中的session给所有java的类使用

  

### 2.1 vue脚手架之vue-cli

全局安装vue-cli组件（只需要装一次）

```js
npm install -g @vue/cli
```



视图方式安装vue脚手架vue-cli

```js
vue ui
```



注：bable：将es6转成es5的工具



### 2.2 使用vuex

需求：视图将数据放入store全局数据中，并从store中获取全局数据进行展示



#### 2.2.1 vuex全局存储store数据


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



#### 2.2.2 view视图使用store中数据

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



#### 2.2.3 自定义组件

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



