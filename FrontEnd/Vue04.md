---
title: Vue前台PC项目其一
tags: 
    - Vue2
categories: 
    - FrontEnd
abbrlink: 33425
date: 2022-05-20 15:22:56
summary: vue2入门项目 —— 电商平台
description: vue2入门项目 —— 电商平台
---

### Vue前台PC

- 来源b站尚硅谷[点我跳转](https://www.bilibili.com/video/BV1Vf4y1T7bw?spm_id_from=333.337.search-card.all.click)
- 项目以实现目标为主
- 优化留到最后

##### 问题汇总
1. vue版本 2.6.11,如果没有`vue-template-compiler`需要安装（用于 模板编译）
2. eslint错误级别禁用 在vue.config.js中 `lintOnSave:false` ,
3. 引入 less预编译器,让vue能够识别less


```css
   <style lang="less" scoped>

   </style>
```

#### 项目文件夹

> public
> src
>
> > api							//发送请求相关
> >
> > > ajax.js								//axios二次封装
> > >
> > > index.js								//包含应用的所有接口的接口请求函数
> > >
> > > mockAjax.js						//mock模拟接口函数
> >
> > assets						//外部资源
> > components				//普通组件
> > mock							//mock模拟接口
> > pages						//路由组件
> > plugins						//插件
> >
> > > element.js							//element-ui插件		
> > >
> > > swiper.js								//swiper轮播图
> > >
> > > vaildate.js								// 表单验证插件的配置文件
> >
> > router						//注册路由相关
> >
> > > index.js									//路由器对象
> > >
> > > routes.js									//所有路由匹配的数组
> >
> > store						//vuex
> >
> > > modules									//各 组件小模块
> > >
> > > index.js										//vuex最核心的管理对象store 
> >
> > utils						//工具包
> > App.js					//App
> > main.js					//主函数
>
> package.json
> vue.config.js

### 开始工作
#### 观察页面确定页面主体框架

- 所有的功能页面都是 上中下结构    上和下是不变化的，只有中间在变化  
Home组件 结构[点我查看Home组件结构](https://shinoimg.yyshino.top/img/vue04_construction.jpg)

- **第一个大组件 Home**
- **组件  由上往下**

> Header
> Home
>
> > TypeNav
> > ListContainer
> > TodayContainer
> > Rank
> > Like
> > Floor
> > Floor
> > Floor
>
> Footer

#### 拆分组件

- ##### 拆分静态组件 ==> 普通组件和路由组件（）

  1. 普通组件  ==> 局部注册  或  全局注册(main.js中注册)
  2. 路由组件  ==> 
     1. 在routes 当中注册
     2. `<a>`>标签 替换为 `<router-link>`
     3. `router-view`选定路由组件展示区域

#### 获取数据

1. **二次封装axios**

`ajax.js`
```js
   /* 
   axios二次封装

   1. 配置通用的基础路径和超时
   2. 显示请求进度条
   3. 成功返回的数据不再是response, 而直接是响应体数据response.data
   4. 统一处理请求错误, 具体请求也可以选择处理或不处理
   */

   import axios from "axios";
   import NProgress from "nprogress";
   import 'nprogress/nprogress.css'
   import store from '@/store'

   /*1. 配置通用的基础路径和超时 */
   // service是一个能任意ajax请求的函数，当然可以作为对象使用
   const service = axios.create({
      // http://gmall-h5-api.atguigu.cn/api/product/getBaseCategoryList
      baseURL:'http://gmall-h5-api.atguigu.cn/api',  // 基础路径
      timeout:20000,  //超时事件
   })

   // 添加请求拦截器
   service.interceptors.request.use((config)=>{
      /* 2.显示请求进度条 */
      // 显示请求进度条: 在请求拦截器中
      NProgress.start()

      let userTempId = store.state.user.userTempId
      if(userTempId){
         config.headers.userTempId = userTempId
      }

      // 携带登录后标识token  
      let token = store.state.user.token
      if(token){
         config.headers.token = token
      }

      // 必须返回config
      return config   //后面会根据返回的config,使用xhr对象发ajax请求
   })

   // 添加响应拦截器
   service.interceptors.response.use(
      response => {   //请求成功的返回的回调
         // 隐藏请求进度条: 在响应拦截器的成功的回调中
         NProgress.done()
         
         /* 成功返回的数据不再是response，而直接是响应体数据response.data */
         return response.data
      },
      error => {    //请求失败的返回的回调
         // 隐藏请求进度条: 在响应拦截器失败的回调中
         NProgress.done()

         /* 4.统一处理请求错误,具体请求也可以选中处理或不处理 */
         alert(error.message || '未知的请求错误')
         // return new Promise(() => {})

         // return error //  不能这么写
         // throw error      //抛出错误
         return Promise.reject(error)   //返回一个失败的Promise, 并将错误传递下去
      }
   )

   // service.get('./xxx').then((result) => { 
   //     // const result = response.data

   //  }).catch(( error ) => { 
   //     // 做一些提示之外的特定工作
   //   })

   // 向外暴露 service
   export default service
```

   2. **阅读接口文档**
   3. **postman工具测试接口**
      1. postman是用来测试API接口的工具
      2. postman也是一个活接口文档
      3. 使用步骤
         (1) 启动 \==\=> 选择登陆\=\=> cancel \=\==> 进入主界面
         (2) 输入url/参数进行请求测试
         (3) 注意post请求体参数需要指定为json格式
         (4) 保存测试接口 ==> 后面可以反复使用
   4. **接口函数**
      - 所有接口的请求函数模块，我们定义一个index.js去写
	   - 以后请求什么数据直接导入去调函数就可以 
         示例
         `index.js`

```js
   /* 
   包含所有接口请求函数的模块
   */
   import ajax from './ajax'

   //获取商品的三级分类列表
   export const reqBaseCategoryList = ()=>ajax.get('/product/getBaseCategoryList')
```
   5. **发送请求** ==> 产生跨域
      1. 解决跨域
      vue.config.js
```js
//配置代理服务器
devServer: {
     proxy: {
       '/api': { // 只对请求路由以/api开头的请求进行代理转发
         target: 'http://182.92.128.115', // 转发的目标url
         changeOrigin: true // 支持跨域
         // pathRewrite: {‘^/api’: ‘’}
       }
     }
```

   6. **是否用到 `vuex`管理数据**

      1. vuex
         - 以home组件为例
         - 在store中 异步ajax获取数据
         - `store/modules/home.js`  ( 如下代码)

         ```js
            /* 
            vuex管理的home模块
            */
            import {reqBaseCategoryList} from '@/api'
         
            const state = {
              baseCategoryList: [], // 所有分类的数组
            }
         
            const mutations = {
              /* 
              接收保存分类列表
              */
              RECEIVE_BASE_CATEGORY_LIST(state, list) {
                state.baseCategoryList = list
              }
            }
         
            const actions = {
              /* 
              异步获取商品三级分类列表
              */
              async getBaseCategoryList({ commit }) {
                const result = await reqBaseCategoryList();
                if (result.code === 200) {
                  commit('RECEIVE_BASE_CATEGORY_LIST', result.data)
                }
              },
            }
         
            const getters = {
         
            }
         
            export default {
              state,
              actions,
              mutations,
              getters
            }
         ```

         - 在组件中

              - dispatch

              - `mapState` 和 `mapGetters`

                ```js
                /*
                mapState,mapGetters是 vuex提供给我们的辅助函数
                	mapState 辅助函数帮助我们生成计算属性
                	mapGetters 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性：
                */
                
                // 必须先引入
                import {mapState,mapGetters} from 'vuex'
                 
                	//计算属性中
                	  ...mapGetters(['detailArrayList']),
                      ...mapState({
                        tradeInfo:state => state.trade.tradeInfo || {},
                        // mock 模拟userAddressList接口
                        userAddressList:state => state.trade.userAddressList || []
                      })
                ```

      2. 不使用vuex

            - 将状态直接定义在当前组件当中
            - 异步ajax发送获取数据
      
   7. **mock模拟数据接口**

      -  安装 `npm install mockjs`
      -  文档
         - [mock-doc](http://mockjs.com/)
         - [mock-github](https://github.com/nuysoft/Mock)
      -  web前台
         - 后台向前台提供API接口, 只负责数据的提供和计算，而完全不处理展现
         - 前台通过Http(Ajax)请求获取数据,　在浏览器端动态构建界面显示数据
      -  理解JSON数据结构
         a. 结构: 名称, 数据类型
         b. value
         c. value可以变, 但结构不能变
      - 利用axios 封装mock 接口函数
      - mockAjax与ajax只在基础baseUrl上有区别

      ```js
         // 只需要将原来的替换 即可
         baseURL:'http://gmall-h5-api.atguigu.cn/api'
         ==>
         baseURL:'/mock'
      ```

         - 在mockSever.js中 书写接口函数（格式如下）


```js
/* 
利用mockjs提供mock接口
*/

import Mock from 'mockjs'
import recommends from './recommends.json'
import floors from './floors.json'
import ranks from './ranks.json'
import likes from './likes.json'
import banners from './banners.json'
import address from './address.json'

// 提供今日接口
Mock.mock('/mock/recommends',{code:200,data:recommends})

// 提供楼层接口
Mock.mock('/mock/floors',{code:200,data:floors})

Mock.mock('/mock/ranks',{code:200,data:ranks})

Mock.mock('/mock/likes',{code:200,data:likes})

Mock.mock('/mock/banners',{code:200,data:banners})

Mock.mock('/mock/address',{code:200,data:address})

console.log('MockServer()');
```

#### 展示数据

- 普通数据

  ```js
  v-if	v-for	:(v-bind)	@(click)
  ```

- 需传递数据

  ```js
  //组件通信
  //1. props 组件通信的方式
  	//父 => 子
  	//父组件中
  	<Xxx :data= "data"/>
      //子组件接收数据
      props:{
          data:数据类型
      }	
  
  
  //2. vue自定义事件
  	// 子 => 父
  	// 在当前组件绑定自定义事件
  	mounted(){
         // xxxXxx为分发的事件
         this.$xxx.$on('xxxXxx',this.xxxXxx)
         // 使用 $once只能触发一次
         // this.$xxx.$once('xxxXxx',this.xxxXxx)
      }
  	// 在子组件当中适当的位置去触发事件并传递参数
  	// $emit(),在子组件当中去触发，子组件对象触发
  	// xxx 为传递的数据
  	this.$emit('addComment',xxx)
  	
  
  //3. 通信方式 全局事件总线
  	// 任意情况
  	// 两个条件
  		// - 1、所有的组件对象都能找到它
  		// - 2、可以调用$on和$emit
  	// 在主函数 vue 的对象中
  	   beforeCreate(){
          // 1) 创建或指定事件总线对象，保存到Vue的原型上
          Vue.prototype.$bus = this
      }
  	//剩余操作 同 2
  
  	
  	
  
  //4. 通信方式 slot插槽
  	// 组件可多次复用时使用
  
  //消息订阅(subscribe)与发布(publish) [一般不用，因为有全局事件总线]
  ```
  

#### 中点站

##### 工作流程

拆分静态组件 => 通过ajax.js书写接口函数获取数据 => 利用vuex管理数据(数据少时也可以直接在当前组件中) => 自定义事件/通过vue指令 => 传递/渲染数据到页面

- 通过上述的工作流程，我们已经可以解决绝大多数问题

- 由组件的视角我们需要

  - Footer组件以及Header组件（ajax + vuex + v-for）

  - Home的部分子组件

    - `TypeNav`

      ```js
      //TypeNav需要在 Home以及Search 下都需要使用 => 在main.js中定义为全局组件
      //引入
      import TypeNav from '@/components/TypeNav'
      // 注册全局组件
      Vue.component(TypeNav.name,TypeNav)
      ```

      ```js
      // 当点击TypeNav时我们需要跳转到搜索页面(携带参数),并且隐藏一级列表
      
      //在data中定义标识 数据 currentIndex isShowFirst
      currentIndex:-2 // 默认为-2  表示当前选中的商品分类 商品分类有0-15
      isShowFirst:path === '/' // bool 表示当前是否为 首页  
      //path为vue-router 指定的当前路径this.$route.path
      
      //定义方法改变
       	/* 
          隐藏一级列表
          */
          hideFirst(){
            // 标识当前已经离开包含分类div了
            this.currentIndex=-2
            // 如果当前不是首页，隐藏一级列表
            if(this.$route.path !== '/'){
              this.isShowFirst = false
            }
          },
      
          /* 显示一级列表 */
          showFirst(){
            // 标识当前已经进入包含分类div了
            this.currentIndex = -1
            //保证显示一级列表
            this.isShowFirst = true
          }
      
          /* 
          显示指定下标的子分类列表
          */
          // showSubList:_.throttle((index) => {  //不可以，原因在于箭头函数没有自己的this，且不能通过bind来指定特定的this
          showSubList:throttle(function(index) {  //这个事件监听回调函数调用的频率太高
            console.log('throttle',index);
            // 只有当还没有离开整个分类的div是才更新下标
            if(this.currentIndex !== -2)
              this.currentIndex = index;
          },60,/* {
            trailing:'false'  //最后一次事件不延迟处理
          } */)
      
      // 定义搜索方法
      toSearch(event) {
            const target = event.target;
            console.dir(target);
            const { categoryname, category1id, category2id, category3id } = 
              target.dataset;
      
            // if(target.tagName.toUpperCase() === 'A'){
            if (categoryname) {
              //准备query参数
              // categoryName=图书、影像、电子书刊&category1Id=1
              const query = {
                categoryName: categoryname,
              };
              if (category1id) {
                query.category1Id = category1id;
              } else if (category2id) {
                query.category2Id = category2id;
              } else if (category3id) {
                query.category3Id = category3id;
              }
      
              // 准备一个用跳转的对象
              const location = {
                name:'search',
                query,
                params:this.$route.params  //需要携带上当前已有params参数
              }
      
              // 跳转到search
              /* 
              从其他到搜索页：push()
              从搜索到搜索页：replace()
              */
             if(this.$route.name === "search"){ //当前是搜索
                this.$router.replace(location)
             }else{
               this.$router.push(location);
             }
              // 隐藏一级列表
              this.hideFirst()
            }
      }
      ```

    - `ListContainer`

      ```js
      // banner轮播图数据 由接口获取
      
      // template数据展示
                <swiper :options="swiperOptions">
                    <swiper-slide class="swiper-slide" v-for="banner in bannerList" :key="banner.id">
                        <img :src="banner.imageUrl" />
                    </swiper-slide>
                  <div class="swiper-pagination" slot="pagination"></div>
                  <div class="swiper-button-prev" slot="button-prev"></div>
                  <div class="swiper-button-next" slot="button-next"></div>
                </swiper>
      
      
      // 利用 swiper 展示轮播图
      // data中定义
            swiperOptions:{
                // direction: 'horizontal', // 垂直切换选项
                loop: true, // 循环模式选项
                autoplay:{  //自动轮播
                  delay:4000,
                  disableOnInteraction:false    //用户操作后是佛停止自动播放
                },
      
                // 如果需要分页器
                pagination: {
                  el: '.swiper-pagination',
                },
      
                // 如果需要前进后退按钮
                navigation: {
                  nextEl: '.swiper-button-next',
                  prevEl: '.swiper-button-prev',
                },
              }
              
      ```

      具体可查看[swiper官网](https://www.swiper.com.cn/)

    - `TodayRecommend` `Rand` `Like` `Floor` 后台获取数据 => 展示(v-for)

