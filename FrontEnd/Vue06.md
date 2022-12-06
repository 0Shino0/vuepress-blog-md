---
title: Vue前台PC项目其三
tags: 
    - Vue2
categories: 
    - FrontEnd
abbrlink: 24151
date: 2022-05-23 08:40:37
summary: vue入门项目 —— 电商平台
description: vue入门项目 —— 电商平台
---



### 优化相关

#### swiper相关

##### swiper轮播图影响多个页面

- 通过选择器可以指定哪个地方需要，但是不好
- 通过ref最好



##### swiper创建的时间应该是在页面列表创建之后才会有效果

- 静态页面是没问题的

- 静态页面不需要等待数据，因此monted完全可以去创建swiper

- 现在我们的数据是动态的，monted内部去创建，数据还没更新到界面上，因此无效
- 可以使用延迟定时器去创建 但是不好



##### 使用watch + nextTick  去解决比较好	

- ​	Vue.nextTick 和 vm.$nextTick 效果一样

- ​	nextTick是在最近的一次更新dom之后会立即调用传入nextTick的回调函数

  ```js
    /* 
    在列表数据已经有了,且已经更新显示了
    数据变化后  ==>  同步调用监视的回调  => 最后异步更新界面
    watch:监视bannerList，就可以知到有数据了
    nextTick:界面更新后执行回调
    */
    watch:{
      bannerList(){   //此时只是数据有了, 但界面还没用更新
        // swiper对象必须在列表显示之后穿件才有效果
  
        /* 
        Vue.$nextTick(callback)
        在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。
        */
        this.$nextTick(() => {    //在此次数据变化导致界面更新完成后执行回调
           // DOM 更新了
          new Swiper (this.$refs.swiper, {
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
          })
        })
  
      }
  ```



##### Floor当中的轮播没效果？

- ​	它是根据数据循环创建组件对象的，外部的floor创建的时候
- ​	所以数据肯定是已经获取到了，所以我们在mounted内部去创建swiper

##### 定义可复用的轮播组件

- ​	banner是在watch当中去创建swiper 因为组件创建的时候数据不一定更新
- ​	floor是在mounted当中去创建swiper，因为内部组件创建的时候，数据已经存在了



#### Search优化

##### 根据分类和关键字进行搜索，解决在search组件内部再进行搜索的bug

```js
//真正到了搜索页面我们都要去解析拿到相关的参数  修改我们的搜索参数
	//beforeMount 去同步更新data数据

	//mounted     去异步发送请求

//在搜索页重新输入关键字或者点击类别不会再发送请求，因为mounted只会执行一次，需要监视路由变化
```



##### 动态显示和删除选中的搜索条件发送请求

- ​	判断参数内部是否存在categoryName  存在就显示
- ​	判断参数内部是否存在keyword 存在就显示
- ​	点击事件，如果删除就把参数对应的数据清除，顺便发送新的请求



##### 解决删除选中的搜索条件后路径不变的bug

- ​	上面删除发送请求我们的请求路径还是不变
- ​	我们需要手动去push跳转到去除对应参数的新路由

```js
// 在 Search组件 中的 removeCategory()和 removeKeyword()方法中添加
          // this.$router.push({  //有历史记录
          this.$router.replace({	// 无历史记录
            name:'search',
            // params:this.$route.params,
            query:this.$route.query
          })
```



##### 响应式对象数据属性的添加和删除

```js
对象当中的属性数据更改会导致页面更改，响应式数据

添加：
	错的：如果对象当中没有对应的属性数据： 直接添加一个属性，这个属性不是响应式的
		因为vue只是在开始对对象当中的所有属性添加getter和setter，后期直接添加的没有
	
	对的：我们需要使用Vue.set  this.$set方法  这样的添加属性就是响应式的   必须对响应式对象添加属性

删除：
	错的： 直接delete删除对象当中的属性，不会导致页面更改
		因为响应式属性只是在检测属性值的改变而不是检测属性的删除
 
	对的：我们需要使用Vue.delete this.$delete方法  除了删除，还添加了更新界面的操作
```



