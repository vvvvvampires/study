# ewshop

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


# 首页
## 导航制作
1.先在通用路由文件index.js里设置底部导航的路由
```
const Home = () => import('../views/home/Home')
const routes = [
    {
    path: '/home',
    name: 'Home',
    component: Home,
    meta:{
      title:'图书兄弟'
    }
  }
```

2.在App.vue里设置
```
<router-link class="tab-bar-item" to="/">
      <div  class="icon"><i class="iconfont icon-shouye"></i></div> 
      <div>首页</div>
</router-link>
```

3.修改导航栏样式

## 标题栏制作
1.在通用组件里构建  NavBar.vue
使用插槽，插槽用于决定将所携带的内容，插入到指定的某个位置，从而使模板分块；插槽显不显示、怎样显示是由父组件来控制的，而插槽在哪里显示就由子组件来进行控制。

slot是对组件的扩展，通过slot插槽向组件内部指定位置传递内容，通过slot可以父子传参；是“占坑”，在组件模板中占好了位置，当使用该组件标签时候，组件标签里面的内容就会自动填坑（替换组件模板中< slot >位置），当插槽也就是坑< slot name=”mySlot”>有命名时，组件标签中使用属性slot=”mySlot”的元素就会替换该对应位置内容；

2.将标题栏分为3部分，加上具名插槽
```
<!-- 左部分 -->
<div class="left" @click="goback">
    <slot name="left">
        <img src="~assets/images/left.png" alt="">
    </slot>
</div>
<!-- 中间部分 -->
<div class="center"><slot>EWShop</slot></div>
<!-- 右部分 -->
<div class="right"><slot name="right"></slot></div>
```

3.在需要用到插槽的地方，导入组件，注册组件，再使用
```
<nav-bar>
    <template v-slot:default>图书兄弟</template>
</nav-bar>
```

4.导航守卫
```
router.beforeEach((to, from, next) => {
　　console.log(store.state.token)
// to: Route: 即将要进入的目标 路由对象
// from: Route: 当前导航正要离开的路由
// next: Function: 一定要调用该方法来 resolve 这个钩子。执行效果依赖 next 方法的调用参数。
　　const route = ['index', 'list'];
　　let isLogin = store.state.token; // 是否登录
// 未登录状态；当路由到route指定页时，跳转至login
　　if (route.indexOf(to.name) >= 0) {
　　　　if (isLogin == null) {
　　　　　　router.push({ path: '/login', });
　　　　}
　　}
// 已登录状态；当路由到login时，跳转至home 
　　console.log(to.name)
　　localStorage.setItem('routerName', to.name)
　　if (to.name === 'login') {
　　　　if (isLogin != null) {
　　　　　　router.push({ path: '/HomeMain', });;
　　　　}
　　}
　　next();
});
```

## 首页推荐商品组件
1.做一个banner滚动，样式

2.做推荐导航栏。新建一个组件，在Home.vue里面加入该组件

3.从Home.vue获取数据，通过属性存起来
```
 getHomeAllData().then(res=>{
        /* 只要首页中推荐商品的数据goods */
        recommends.value = res.goods.data;
        banners.value = res.slides;
 })
```
4.RecommendView.vue接受属性，才可以用到Home.vue传过来的数据
```
/* 接受属性，才可以用到数据 */
props: {
      recommends: {
            type:Array,
            default() {
                 return [];
            }
      }
}
```

5.点击推荐的图书，跳转到商品详情页。
```
/* 跳转到商品详情页 */
setup(){
    /* 申明路由器 */
    const router = useRouter();

     /* 跳转到商品详情页 */
     const goD = (id) =>{
            router.push({path:'/detail', query:{id}})
     }

     return {
          goD
     }
}
```
## 首页选项卡组件
1.注册一个新的选项卡组件

2.在Home.vue中引入组件

3.设置选项卡的几个titles,在TabControl.vue接受titles：props

4.设置选项卡的点击，选择哪个，当前的index就会变成哪个，就会高亮。
```
  <!-- :classs 有个激活的属性 -->
  <div v-for="(item, index) in titles" :key="index" 
       @click="itemClick(index)"
       :class="{active:index === currentIndex}"  
       class="tab-control-item">
    <span>{{item}}</span>
  </div>

  setup(props, {emit}){
     let currentIndex = ref(0);
     /* 点击选项卡，选择哪个，当前的index就会变成哪个 */
     const itemClick = (index) => {
       currentIndex.value =index;
       emit('tabClick', index)
     }
     /* 返回方法 */
     return{
       currentIndex,
       itemClick
     }
  }
```

5.点击选项卡，下方出现对应的内容。子组件设置点击函数，之后传给父组件
```
emit('tabClick', index)
```
父组件调用子组件传递过来的方法
```
<tab-control @tabClick="tabClick" :titles="['畅销', '新书', '精选']">
</tab-control>

const tabClick = (index)=>{

                let types = ['sales', 'new', 'recommend'];

                currentType.value = types[index];

                nextTick(()=>{
                    // 重新计算高度
                    bscroll &&  bscroll.refresh();
                })
}
```

## 首页商品列表组件设计和开发
1.商品项和商品列表是两个不同的组件，在商品列表里引入一个一个商品项

2.设置每个商品的样式

3.获取数据

①.vue获取列表数据；设置商品数据模型，加载动态数据；通过计算属性，设置不同类型列表的数据，如新书、推荐、销售列表；
```
/*路由： 获取首页商品数据 */
export function getHomeGoods(type='sales', page='1'){
  return request({
    /* 默认请求的是销售排行的数据，页数是1 */
    url:'/api/index?'+type+'=1&page='+page
  })
}
                /* 按销售排行的数据 */
                getHomeGoods('sales').then(res=>{
                    goods.sales.list = res.goods.data;
                })

           // 商品列表数据模型，加载动态数据
            const goods = reactive({
                sales:{page:1, list:[]},
                new: {page:1, list:[]},
                recommend: {page:1, list:[]}
            })

            /* 设置当前默认的数据列表 销售 */
            let currentType = ref('sales');

            //使用ref后要用.value才能把数据获取出来
            const showGoods = computed(()=>{
                return goods[currentType.value].list;
            })
```

②GoodsList通过props接收首页数据
```
  /* 属性接收 */
  props:{
    goods:{
      type:Array,
      default(){
        return []
      }
    }
  }
  ```
③GoodsList再通过 :product="item" 属性 传递给子组件GoodsListItem
```
 <!-- 再通过:product="item"属性 传递给子组件GoodsListItem -->
<GoodsListItem v-for="item in goods" :product="item" :key="item.id">
</GoodsListItem>
```

④Home.vue切换不同的商品列表数据：只需要改变类型，页数就能切换列表
```
/* 子组件传递过来的，点击切换不同的商品列表数据 */
            const tabClick = (index)=>{

                let types = ['sales', 'new', 'recommend'];

                currentType.value = types[index];

                nextTick(()=>{
                    // 重新计算高度
                    bscroll &&  bscroll.refresh();
                })
            }
```

4.上拉加载更多数据:betterScroll组件
创建BetterScroll对象
```
bscroll = new BScroll(document.querySelector('.wrapper'), {
    probeType: 3,  // 0, 1, 2, 3, 3 只要在运动就触发scroll事件
    click: true, // 是否允许点击
    pullUpLoad: true //上拉加载更多， 默认是false
});
```

## 回到顶部组件 keep-alife使用
回到顶部组件调用Home.vue组件中的btop方法
```
/* emit子组件调用父组件的btop */
setup(props, {emit}){
    return{
      backtop:()=>{
        emit('bTop');
      }
    }
  }

Home.vue中：
const bTop = () =>{
                /* 滚动到0，0，500毫秒 */
                bscroll.scrollTo(0, 0, 500);
}
```
keep-alife保存生命的方法，切页面的时候不用重新渲染保持页面，可以缓存，在App.vue组件中使用

## vant组件库
1.使用swiper 轮播图
HomeSwiper接受属性

2.使用图片懒加载和徽章
```
懒加载：
<img v-lazy="item.img_url" alt="">
徽章：
<van-badge :content="$store.state.cartCount" max="9">
    <img alt=""  src="./assets/images/trolley.svg"/>
</van-badge>
```

# 分类页面
## 布局和菜单
1.侧边栏导航组件，折叠面板
```
      <van-sidebar class="leftmenu" v-model="activeKey">
        <van-collapse v-model="activeName" accordion>
          <van-collapse-item v-for="item in categories" :key="item.id"
                             :title="item.name"
                             :name="item.name">
            <!-- 遍历子级 -->
            <van-sidebar-item
                v-for="sub in item.children"
                :title="sub.name"
                :key="sub.id"
                @click="getGoods(sub.id)"
            />

          </van-collapse-item>

        </van-collapse>
      </van-sidebar>
```

2.分类页面交互
导航栏、商品卡片，选中一一对应，获取id
```
const getGoods = (cid) =>{
      currentCid.value = cid;
      init();
      console.log("当前分类id: "+currentCid.value);
      console.log("排序的序号："+currentOrder.value);
    }
```

3.获取接口数据
数据模型
```
    //数据模型
    const goods = reactive({
      sales:{page:0, list:[]},
      price:{page:0, list:[]},
      comments_count:{page:0, list:[]}
    })
```

4.上拉加载更多数据

## 获取商品详情信息
分类页面、首页的列表
```
      /* 点击查看详细信息，做路由 */
      itemClick:(id)=>{
        console.log(id);
        router.push({path:'/detail', query:{id}});
      }
```
详细页面,获取详细信息
```
    onMounted(()=>{
      /* 接受商品id */
      id.value =route.query.id;
      getDetail(id.value).then(res=>{
        book.detail = res.goods;
        book.like_goods = res.like_goods;
      })
    })
```

## 渲染商品数据到模板中
图片组件、商品卡片组件、按钮、标签组件（概述、热评、相关读书）

#  登录注册
购物车、订单要登录才能操作

## 用户注册组件
表单组件，用户信息与组件相关联，做验证再提交给服务器；

验证时,密码不一致：提示组件

提交失败，在请求request.js中偶同意提示错误信息，不然有多个地方会提示，麻烦

## 用户登录
1.登陆成功，将token保存到本地Windows.localStorage
```
window.localStorage.setItem('token', res.access_token)
```

2.用户登录授权方案处理
```
  // 请求拦截，不带token不让过
  instance.interceptors.request.use(config=>{
    // 如果有一个接口需要认证才可以访问，就这里统一设置
    const token = window.localStorage.getItem('token')
    if (token){
      config.headers.Authorization = 'Bearer '+ token;
    }
    // 直接放行
    return config;
  }, err=>{

  })
  ```

3.退出登录
```
    const toLogout = () => {
      logout().then(res => {
        Toast.success('退出成功')
        /* 退出后，清楚token */
        window.localStorage.setItem('token', '')
       /* <!-- 跳转到登录页面 --> */
        setTimeout(() => {
          router.push({path: '/login'})
        }, 500)
      })
    }
```

4.转为状态管理
```
/* 使用状态 */
import {useStore} from 'vuex'


/* 调用状态方法 */
store.commit('setIsLogin', true)

我的、购物车需要授权登录，在路由处设置
{
    path: '/profile',
    name: 'Profile',
    component: Profile,
    meta:{
      title:'我的',
      isAuthRequired:true/* 需要授权登录 */
    }
```

# 购物车
## 添加购物车
1.设置路由
cart.js添加购物车、修改购物车、选择商品状态、获取购物车列表、移出购物车路由

2.在Detail.vue中写添加购物车、立即购买的方法

3.购物车列表和数量改变操作
让页面保持购物车之前的数量
```
setup() {
    /* 状态管理 */
    const store = useStore();
    /* 当页面一加载的时候，执行更新购物车的值 */
    onMounted(()=>{
    store.dispatch("updateCart")

    })
  }
```

4.购物车页面数据的显示：数据申明
①购物车为空，前往购物，前往首页
②购物车有多少数据，就一一遍历 展现在购物车列表里
③修改购物车的数量

5.改变购物车选中的状态
复选框组件
全选

## 删除购物车商品和计算总价
1.删除
调用接口里deleteCartItem()方法，删除后重新加载购物车

2.计算总价
通过计算属性 计算总价
选中的计算，数量加在一起

3.创建订单
```
const onSubmit = () => {
      /* 用户没有选中的结算商品 */
      if(state.result.length == 0) {
        Toast.fail("请选择商品进行结算");
        return;
      }else{
        /* 到创建订单的页面去 */
        router.push({path:'/createorder'});
      }
    }
```

## 个人中心
1.获取到用户信息

2.跳转其他页面方法

## 地址管理
1.管理省市县 第三方插件  从网上下载一个列表 utils

2.封装地址的网络请求
```
// 添加地址的网络请求
export function addAddress(params) {
    return request({
        url:'/api/address',
        method: 'post',
        params
    })
}
```

3.添加地址列表
选中默认的地址，切换地址、新增地址
①构造省市区 列表

②获取所有填写参数

③添加数据

4.地址管理列表
获取列表数据、显示地址的信息
```
    onMounted( () => {
        /* 获取列表数据 */
        getAddressList().then(res=>{
            if(res.data.length == 0) {
              state.list = [];
              return;
            }

            state.list = res.data.map(item=>{
              /* 显示的地址信息 */
              return {
                  id:item.id,
                  name:item.name,
                  tel:item.phone,
                  address:`${item.province} ${item.city} ${item.county} ${item.address}`,
                  isDefault: !!item.is_default
              }
            });

        })
    })
```

5.编辑地址信息

# 订单
### 订单
1.写订单接口
创建订单、获取定单预览、定单支付, 获取二维、定单状态、获取定单列表 {page:1, status:2, include:'user,orderDetail.goods'}、定单详情、确认定单、获取物流信息 /api/orders/{order}/express

2.订单预览

3.计算总和

## 创建订单
1.创建订单会把购物车里的数据删除掉

## 订单支付流程
1.弹出层组件

2.获取支付信息（二维码）

3.使用九宫格放二维码信息

4.轮询查看，等待用户的支付信息

5.支付成功，跳转到订单详情页面

6.不支付关闭，也可以到订单详情 

## 订单详情处理
订单详情、订单状态

## 订单列表
上拉刷新 加载更多 List组件

下拉刷新更多

打包
