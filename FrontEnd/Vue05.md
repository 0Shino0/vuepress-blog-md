---
title: Vue前台PC项目其二
tags: 
    - Vue2
categories: 
    - FrontEnd
abbrlink: 15766
date: 2022-05-22 12:24:41
summary: vue2入门项目 —— 电商平台
description: vue2入门项目 —— 电商平台
---

### Vue前台Pc项目其二

获取`ajax` `vuex` `vue-router`数据等在上一篇[Vue前台Pc项目其一](https://yyshino.top/posts/33425.html)

#### Search组件

#### 分析Search组件的结构

Search组件 [点我查看Search组件](https://shinoimg.yyshino.top/img/vue05_search.jpg)

> Search的头部 (关键字展示区)
>
> > SearchSelect (关键字选择区)
>
> Search的主要部分 (商品详细区)
>
> > MyPagination (页码区)

##### 初始化获取数据

```js
// 首先 在data中初始化数据
    data(){
      return{
        options:{
          category1Id: '', // 一级分类ID
          category2Id: '', // 二级分类ID
          category3Id: '', // 三级分类ID
          categoryName: '', // 分类名称
          keyword: '', // 搜索关键字
  
          // trademark: '', // 品牌: "ID:品牌名称" "1:苹果"
          props: [], // 商品属性的数组: ["属性ID:属性值:属性名"] ["2:6.0～6.24英寸:屏幕尺寸"]
          order: '1:desc', // 排序方式  1: 综合,2: 价格 asc: 升序,desc: 降序  "1:desc"
  
          pageNo: 1, // 页码
          pageSize: 10, //	每页数量
        }
      }
    }

// 异步获取 商品数据
        /* 
        异步获取商品列表
        */
        getShopList(page=1){
          // 更新options中的pageNo
          this.options.pageNo = page
          // 发送搜索请求
          this.$store.dispatch('getProductList',this.options)
        }

// methods和computed中 获取vuex管理的接口数据
        getShopList(page=1){
          // 更新options中的pageNo
          this.options.pageNo = page
          // 发送搜索请求
          this.$store.dispatch('getProductList',this.options)
        }

...mapGetters(['goodsList','total']) // 注意需要引入 mapGetters
```



```js
// 管理我们的关键字
// 利用 v-if 选择展示 关键字
          <ul class="fl sui-tag">
            <li class="with-x" v-if="options.categoryName">{{options.categoryName}}
              <i @click="removeCategory">×</i>
            </li>
            <li class="with-x" v-if="options.keyword">{{options.keyword}}
              <i @click="removeKeyword">×</i>
            </li>
            <li class="with-x" v-if="options.trademark">{{options.trademark}}
              <i @click="removeTrademark">×</i>
            </li>
            <li class="with-x" v-for="prop in options.props" :key="prop">{{prop}}
              <i @click="removeProp(index)">×</i>
            </li>
          </ul>


// 定义 设置和清除关键字的方法
      /* 
      删除一个属性条件
      */
      removeProp(index){
        // 删除props中index的元素
        this.options.props.splice(index,1)
        // 重新请求获取数据列表
        this.getShopList()
      },
      
      /* 
      添加一个属性条件 
      */
      addProp(prop){
        const {props} = this.options
        // 如果已经存在条件数组中，不添加
        if(props.includes(prop)) return
        // 向props数组添加一个条件字符串  子向父通信==>vue自定义事件
        this.options.props.push(prop)
        // 重新请求获取数据列表
        this.getShopList()
      },

      /* 
      删除品牌条件
      */
      removeTrademark(){
        //  更新options中的trademark为指定的值
        // this.options.trademark = ''
        // delete this.options.trademark   // ==> 不会导致界面更新
        this.$delete(this.options,'trademark')
        // 重新请求获取数据列表
        this.getShopList()
      },

      /* 
      设置品牌条件
      */
      setTrademark(trademark){
        //  如果当前品牌已经在条件中了，直接结束
        if(trademark===this.options.trademark) return 
        
        //  更新options中的trademark为指定的值
        // this.options.trademark = trademark
        this.$set(this.options,'trademark',trademark)
        // 重新请求获取数据列表
        this.getShopList()

      },

        /* 
        删除分类的条件
        */
        removeCategory(){
          // 更新分类相关数据u
          this.options.category1Id = ''
          this.options.category2Id = ''
          this.options.category3Id = ''
          this.options.categoryName = ''
          // 重新发起请求
          // this.getShopList()

          // 重新跳转到search，不再携带删除的条件所对应的参数(query)
          // this.$router.push({
          this.$router.replace({
            name:'search',
            // query:this.$route.query,
            params:this.$route.params
          })
        },
        
        /* 
        删除关键字的条件
        */
        removeKeyword(){
          // 更新分类相关数据u
          this.options.keyword = ''
          // 重新发起请求
          // this.getShopList()
          
          // 重新跳转到search，不再携带删除的条件所对应的参数(params)
          // this.$router.push({
          this.$router.replace({
            name:'search',
            // params:this.$route.params,
            query:this.$route.query
          })

          // 3) 在Search中分发事件
          this.$bus.$emit('removeKeyword')
        },
      

        /* 
        更新options中参数属性
        */
        updateParams(){
          // 取出参数数据
          const {keyword} = this.$route.params
          const {category1Id,category2Id,category3Id,categoryName} = this.$route.query

          // 保存到options中
          this.options = {
            ...this.options,
            keyword,
            category1Id,
            category2Id,
            category3Id,
            categoryName
          }
        }

// 管理排序相关
//在computed中添加
     /* 
      得到包含当前分类项标识(orderFlag)和排序方式(orderType)的数组
      */
     orderArr(){
       return this.options.order.split(':')
     }

// 在methods中添加
      /* 
      设置新的排序搜索
      */
      setOrder(orderFlag){
        // 得到当前的排序项和排序方式
        let [flag,type] = this.orderArr
        // 点击的是当前排序项：只需要切换orderType
        if(orderFlag===flag){
            type = type ==='desc'?'asc':'desc'
          // console.log(type);
        }else{
          // 点击的不是当前排序项： 更新orderFlag为指定的值，orderType更新为desc
          flag = orderFlag
          type = 'desc'
        }
        // 请求获取商品分页列表
        this.options.order = flag + ':' + type
        this.getShopList()
      }
```

##### 分页组件

```js
// MyPagination 组件
// Search父组件传递数据 同时初始化 页码数据
            <MyPagination
              :currentPage = "options.pageNo"
              :pageSize = "options.pageSize"
              :total = "total"
              :showPageNo   = "5"
              @currentChange = "getShopList"
            ></MyPagination>

// 接收数据
        props:{
         currentPage:{
             type:Number,
             default:1
         },
         total:{
             type:Number,
             default:0
         },
         pageSize:{
             type:Number,
             default:10
         },
         showPageNo:{
             type:Number,
             default:5,
            //  要求传的值要是奇数
            validator:function(value){
                return value%2 === 1
            }
         },
     },
     data(){
         return {
             myCurrentPage:this.currentPage,  //初始值由父组件来指定
         }
     }

// 设计算法 换页 
// computed 计算总页数
        totalPages(){
            const {total,pageSize} = this
            return Math.ceil(total/pageSize)
        }

//计算首尾页
        startEnd(){
            let start,end
            const {myCurrentPage,showPageNo,totalPages} = this

            // 计算start
            /* 
            举例子

            myCurrentPage,showPageNo,totalPages
            4               5           8       23[4]56
            start = 4-2
            */
            //   start = myCurrentPage - (showPageNo-1)/2
            start = myCurrentPage - Math.floor(showPageNo/2)
            // 如果myCurrentPage比较小，计算的结果可能小于1   start>=1
            /* 
            myCurrentPage,showPageNo,totalPages
            4               5           8       [1]2345
            */
            if(start<1){
                start = 1
            }

            // 计算end
            /* 
            myCurrentPage,showPageNo,totalPages
            4               5           8       23[4]56
            start = 2
            end = 2 + 5 - 1
            */
            end = start + showPageNo - 1
            // end的最大值为totalPages
            /* 
            myCurrentPage,showPageNo,totalPages
            8               5           8       45[6]78
            start = 6   ==> 4
            end = 10 ==> 超过了totalPages  8
            */
            if(end>totalPages){
                // 修改end 为 totalPages
                end = totalPages
                // 修正start    
                start = end - showPageNo +1
                // 一旦总页数小于最大连续页码数 ==>  start<1
                /* 
                myCurrentPage,showPageNo,totalPages
                5               5           4       12[3]4
                end = 4
                start = 0    ===> 1
                */
                if(start<1){
                    start = 1
                }
            }
            return {start,end}
        }



        /* 
        包含从start到end的数组
        */
        startEndArr(){
            const arr = []
            const {start,end} = this.startEnd
            for(let page=start;page<=end;page++){
                arr.push(page)
            }
            return arr
        }
```

##### 商品详情

```js
//Detail
    components: {
      ImageList,
      Zoom
    },

    data(){
      return {
        skuId:'',
        skuNum:1
      }
    },

    // 我们一般在beforeMount当中去处理数据
    beforeMount(){
      this.skuId = this.$route.params.skuId
    },

    computed:{
      ...mapGetters(['categoryView','skuInfo','spuSaleAttrList']),
      // categoryView(){
      //   return this.$store.getters.categoryView
      // }

      imgList(){
        return this.skuInfo.skuImageList || []
      }
    },

    mounted(){
      this.getSkuDetailInfo()
    },

    methods:{
      getSkuDetailInfo(){
        this.$store.dispatch('getSkuDetailInfo',this.skuId)
      },

      // 排它处理用户点击悬着销售属性值 (谁变绿)
      changeChecked(spuSaleAttr,spuSaleAttrList){
        // 第一步：多有的属性值全部变白
        spuSaleAttrList.forEach(item => item.isChecked = '0')
        // 第二步：点击的这个属性值变绿
        spuSaleAttr.isChecked = '1'
      },

      // 添加购物车按钮点击之后的逻辑
      async addShopCart(){
        // 发请求让后台给数据库的购物车数据表中添加一条数据
        // 根据添加成功还是失败来决定要不要条状到添加购物车成功页面

        let {skuId,skuNum} = this

        // this.$store.dispatch('addOrUpdateCart',{skuId,skuNum})
        // action当中异步函数的返回值，异步函数范沪指一定是promis

        try {
          const result = await this.$store.dispatch('addOrUpdateCart',{skuId,skuNum})
          alert('添加购物车成功')
          
          // 成功需要跳转 编程式导航跳转
          // 因为下个页面  添加购物车成功页面需要  商品购物车成功页面需要  商品数量和商品详情信息

          // 以后如果碰到的数据是简单数据，那么我们考虑路由传参
          // 如果碰到数据是浮渣数据 那么我们考虑存储手段
          // localStorage     setItem  getItem   removeItem  clear
          // sessionStorage   setItem  getItem   removeItem  clear
          // 区别：localStorage是永久存储   sessionStorage浏览器关闭就没了 

          // 序列化   将js数据转化为json数据
          sessionStorage.setItem('SKUINFO_KEY',JSON.stringify(this.skuInfo))
          this.$router.push('/addcartsuccess?skuNum='+ this.skuNum)

        } catch (error) {
          alert(error.message)
        }     

        //const result = await this.$store.dispath('addOrUpdateCart'{skuId,skuNum}) 
        // if(result === 'ok'){  
        // }else{          
        // }

      }
    }
```

放大镜大图与小图

- 交互
  - 图片列表的点击切换样式
  - 图片列表点击大图要跟着切换  组件通信index下标	

- 放大镜
  - ​	鼠标动 
  - ​	遮罩动
    - 求遮罩的位置
    - 设置遮罩的位置
  - ​	大图动
    - 大图反向移动遮罩的位置2倍  

```js
//Zoom组件
    props:['imgList'],
    data(){
      return {
        defaultIndex:0    //显示图片的默认下标s
      }
    },

    mounted(){
      this.$bus.$on('syncDefaultIndex',this.syncDefaultIndex)
    },

    computed:{
      defaultImg(){
        // 1、解决家报错  不能在上面直接写   imgList[defaultIndex].imgUrl
        return this.imgList[this.defaultIndex] || {}
      }
    },
    methods:{
      syncDefaultIndex(index){
        this.defaultIndex = index
      },
      move(event){
        // 添加移动事件。代表鼠标移动

        // 获取鼠标位置
        let mouseX = event.offsetX
        let mouseY = event.offsetY



        // 2、计算遮罩的位置让遮罩动
        let mask = this.$refs.mask        // 蒙版
        let maskX = mouseX - mask.offsetWidth/2
        let maskY = mouseY - mask.offsetHeight/2
        let big = this.$refs.big

        // 限定临界值
        if(maskX < 0){
          maskX = 0
        }else if(maskX > mask.offsetWidth){
          maskX = mask.offsetWidth
        }
        if(maskY < 0){
          maskY = 0
        }else if(maskY > mask.offsetHeight){
          maskY = mask.offsetHeight
        }

        mask.style.left = maskX +'px'
        mask.style.top = maskY +'px'

        // 3、大图动        大图向着鼠标相反方向移动两倍
        big.style.left = -maskX * 2 + 'px'
        big.style.top = -maskY * 2 + 'px'
        

      }
    }

//ImageList
    props:['imgList'],

    data(){
      return{
        defaulIndex:0,  //  默认有橙色框框的图片下标

        swiperOptions:{
          // direction: 'horizontal', // 垂直切换选项
          // loop: true, // 循环模式选项
          // autoplay:{  //自动轮播
          //   delay:4000,
          //   disableOnInteraction:false    //用户操作后是佛停止自动播放
          // },

          // 如果需要分页器
          // pagination: {
          //   el: '.swiper-pagination',
          // },

           slidesPerView: 4,
           slidesPerGroup:3,

          // 如果需要前进后退按钮
          navigation: {
            nextEl: '.swiper-button-next', 
            prevEl: '.swiper-button-prev',
          },
        }
      }  
    },
 
    methods:{
      changeDefaultIndex(index){
        this.defaulIndex = index

        // 通过全局事件总线把选中的index传递给zoom
        this.$bus.$emit('syncDefaultIndex',index)
      },
    }
```

ShopCart

```js
    mounted(){
      this.getCartList()
    },

    methods:{
      // 重新发送请求
      getCartList(){
        this.$store.dispatch('getCartList')
      },
      // 修改购物车中商品数量
      async changeCartNum(shopCart,disNum,flag){
        // 正对输入的数据是最终的商品熟练，我们得转化为变化的量
        if(!flag){
          // 输入正数表示用户 想要的数量
          if(disNum > 0){
            disNum = disNum - shopCart.skuNum
          }else{
            disNum = 1-shopCart.skuNum
          }
        }else{
          // 输入负数
            //针对点击+-的数据，传递过来的是变化的量
            if(disNum + shopCart.skuNum <= 0){
              disNum = 1 - shopCart.skuNum
            }
        }
        try {
          // 吧传递古来的数据全部转化为正确的变化的量之后就可以发起请求
          // 修改数据库数据
          await this.$store.dispatch('addOrUpdateCart',{skuId:shopCart.skuId,skuNum:disNum})
          alert('修改数量成功')
          // 重新发起请求
          this.getCartList()
        } catch (error) {
          alert(error.message)
        }

      },
      // 修改购物车中商品的选中状态
      async updateOne(shopCart){
        try {
          await this.$store.dispatch('updateCartChecked',{skuId:shopCart.skuId,isChecked:shopCart.isChecked?0:1})
          alert('修改状态成功')
          this.getCartList()
        } catch (error) {
          alert(error.message)
        }

      },
      // 删除购物车单个
      async deleteOne(shopCart){
        try {
          await this.$store.dispatch('deleteCart',shopCart.skuId)
          alert('删除成功')
          this.getCartList()          
        } catch (error) {
          alert(error.message)
        }
      },
      // 删除购物车多个
      deleteAll(){
        try {
          this.$store.dispatch('deleteCartAll')
          alert('删除多个成功')
          this.getCartList()
        } catch (error) {
          alert(error.message)
        }
      }

    },
    computed:{
      // mapState使用数组   必须名字相同    数据只能总的state当中的数据才能使用，模块化后不能使用
      ...mapState({
        // shopCartList:state => state.shopcart.shopCartList[0].cartInfoList
        /* 
        出现意外问题
        在购物车内刷新出现如下问题
        TypeError: Cannot read properties of undefined (reading 'cartInfoList')
        */
        shopCartList:state => state.shopcart.shopCartList
      }),
      /* 
      选择的数量
      */
      checkedNum(){
        return this.shopCartList.reduce((prev,item) => {
          if(item.isChecked){
            prev += 1
          }
          return prev
        },0)
      },
      /* 
      总共支付
      */
      allMoney(){
        return this.shopCartList.reduce((prev,item) => { 
          if(item.isChecked){
            prev += item.cartPrice * item.skuNum
          }
          return prev
         },0)
      },
      /* 
      判断全选   可读  可写
      */
      isAllCheck:{
        get(){
          return this.shopCartList.every(item => item.isChecked)
        },
        async set(val){
          // this.$store.dispatch('updateCartChecked',cal?1:0)    是调用updateCartCheckedAll异步函数
          // 它的结果拿的是异步函数的返回值   固定的那个promise，不是函数return后面Promise.all的返回值promise
          try {
            const result = await this.$store.dispatch('updateCartCheckedAll',val?1:0)
            alert('修改所有数据成功')
            this.getCartList()
          } catch (error) {
            alert(error.message)
          } 
        }
      },
    }

```

##### 交易订单 Trade

```js
    data(){
      return {
        message:'',
        orderNo:''
      }
    },
    mounted(){
      // 定义函数
      this.getTradeInfo()
      this.getAddress()
    },
    methods:{
      getTradeInfo(){
        this.$store.dispatch('getTradeInfo')
      },
      getAddress(){
        this.$store.dispatch('getAddress')
      },
      // 修改默认收获地址,排它
      changeDefault(userAddress,userAddressList){
        userAddressList.forEach((item => item.isDefault = '0'))
        userAddress.isDefault = '1'
      },

      // 提交订单  不使用vuex
      async submitOrder(){
        // 带上交易编号和交易信息发送请求穿件订单
        let tradeNo = this.tradeInfo.tradeNo
        let tradeData = {
          consignee:this.defaultUserAddress.consignee,
          consigneeTel:this.defaultUserAddress.phoneNum,
          deliveryAddress:this.defaultUserAddress.userAddress,
          paymentWay:"ONLINE",
          orderComment:this.message,
          // 购物车的详细信息 detailArrayList
          orderDetailList:this.detailArrayList
        };
        
        try {
          const result = await this.$API.reqSubmitOrder(tradeNo,tradeData)        
          if(result.code === 200){
            // this.orderNo = result.data    // 这里是是存储订单编号，存不存都行。因为订单编号是要带到下个页面的
            this.$router.push('/pay?orderNo='+result.data)
          }
        } catch (error) {
          alert('提交失败，错误信息：'+error.message)
        }
      }
    },
    /*  */
    computed:{
      // userAddressList 接口损毁
      ...mapGetters(['detailArrayList']),
      ...mapState({
        tradeInfo:state => state.trade.tradeInfo || {},
        // mock 模拟userAddressList接口
        userAddressList:state => state.trade.userAddressList || []
      }),
      // 计算最终确定的用户信息，更具上面点击修改默认而发送变化的
      defaultUserAddress(){
        return this.userAddressList.find(item => item.isDefault === '1') || {}
      }
    }


```

##### 支付Pay

```js
    data(){
      return {
        orderNo:'',
        payInfo:{},
        payStatus:0
      }
    },
    beforeMount(){
      this.orderNo = this.$route.query.orderNo
    },
    mounted(){
      this.getPayInfo()
    },
    methods:{
      async getPayInfo(){
        const result = await this.$API.reqPayInfo(this.orderNo)
        if(result.code === 200){
          this.payInfo = result.data
        }
      },
      // 1、点击按钮能够弹出消息盒子
      // 2、、生成二维码图片（请求回来的支付信息当中包含了一个生成二维码的所有的数据并不是图片url(微信给我们的一个数据)）
      // 3、轮询
      async pay(){
        // 1、点击立即支付后，先生成二维码
          try {
            const imgUrl = await QRCode.toDataURL(this.payInfo.codeUrl)
            // 2、 生成二维码url成功之后，再去弹出消息盒子
            this.$alert(`<img src="${imgUrl}"/>`, '请使用微信扫码支付', {
              dangerouslyUseHTMLString: true,
              showClose:false,
              showCancelButton:true,
              cancelButtonText:'支付遇到问题',
              confirmButtonText:'我已经成功支付',
              center:true,
              beforeClose:(action,instance,done) => {
                if(action === 'confirm'){
                  // 点击确认按钮的逻辑
                  // if(this.payStatus !== 200){
                  //   // success    代表成功   绿色
                  //   // error    代表失败错误   红色
                  //   // warning    代表警告   橙色
                  //   // info       代表提示    灰色 
                  //   this.$message.success('请确保支付成功，支付成功会自动跳转')
                  // }

                  // 后门
                  clearInterval(this.timer)
                  this.timer = null
                  done()
                  this.$router.push('/paysuccess')

                }else if(action === 'cancel'){
                  this.$message.error('请联系前台小姐姐')
                  // 点击取消按钮的逻辑
                  // 清除定时器
                  clearInterval(this.timer)
                  this.timer = null

                  done()
                }
              }
            })
              // .then()   //点击了确认按钮后的操作
              // .catch(); //点击了取消按钮后的操作
              // 无论是点击确认按钮还是点击取消按钮，都会强制的关闭messageBox


                        // 3、设置循环定时器(轮询),发送请求看用户对于这个订单是否支付成功
            if(!this.timer){
              this.timer = setInterval(async () => {
                const result = await this.$API.reqPayStatus(this.orderNo)
                if(result.code === 200){
                  // 4、如果支付成功   
                          // 1)跳转支付成功页面   
                          // 2)清除定时器  
                          // 3)关闭消息盒子  
                          // 4)存储支付成功的状态  是为了后期用户点击按钮的时候的判断依据
                  this.payStatus = 200

                  clearInterval(this.timer)
                  this.timer = null

                  this.$msgbox.close()    //关闭消息盒子

                  this.$router.push('/paysuccess')

                }
              },2000)
            }
          } catch (err) {
            // 生成失败，不需要弹出
            console.error(err)
          }
      }
    }

// 支付成功
          <router-link class="btn-look" to="/center">查看订单</router-link>
          <router-link class="btn-goshop" to="/">继续购物</router-link>
```

##### Login 和 Register

```js
//Login
	data(){
      return {
        phone:'',
        password:''
      }
    },
    methods:{
      async login(){
        // 获取用户数据
        let {phone,password} = this 
        if(phone&&password){
          try {
            // 发请求
            /* 
            13700000000
            111111
            */
           await this.$store.dispatch('userLogin',{phone,password})
           alert('登录成功，准备跳转首页')
          //  下面的需要和导航守卫配合去到想去而没有去到的地方
           let redirect = this.$route.query.redirect || '/'
           this.$router.push(redirect)
          } catch (error) {
            alert(error.message)
          }
        }
      }
    }

//Register
    data(){
      return {
        phone:'',
        code:'',
        password:'',
        password2:'',
        isCheck:''
      }
    },
    methods:{
      async register(){
        const success = await this.$validator.validateAll() // 对所有表单项进行验证
        if(success){
          // 获取用户信息
          let {phone,code,password,password2} = this
          // if(phone && code && password && password2 && password === password2){
            try {
              // 发请求注册用户
                await this.$store.dispatch('userRegister',{phone,code,password})
                alert('注册成功')
                this.$route.push('/login')
            }catch(error) {
              alert(error.message)
            }
          // }

        }


      }
    }
```

