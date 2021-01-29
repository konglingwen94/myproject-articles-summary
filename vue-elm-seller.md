 

项目仓库地址：[https://github.com/konglingwen94/vue-elm-sell](https://github.com/konglingwen94/vue-elm-sell)

项目线上地址: http://123.56.124.33:5000

## 前提

自从学习了`Vue`后，能用`Vue`解决的场景用例最终我都尽可能的用`Vue`去实现。单纯的用例需求并没有完整的项目开发流程，从中能学到的东西也是有限的。在这之前除了使用`Vue`做过[vue-music](https://github.com/konglingwen94/vue-music)的移动端音乐播放器项目和[vue-bytedanceJob](https://github.com/konglingwen94/vue-bytedanceJob)（重构某独角兽互联网公司官方招聘网站）之外，自己并没有用`Vue`涉猎`web`端更复杂的业务场景。

为了找一个项目练习，我去`github`上开始了搜索，当看到`https://github.com/ustbhuangyi/vue-sell`这个项目时，感觉这个移动端应用的一些业务场景是自己没有接触过的，于是我就照着这个应用的`UI`和`功能`用自己的**知识体系**和**技术栈**进行了重构，大概不到半个月的时间，我完成了第一个`commit`提交到项目上线运行，本篇文章就从**应用功能**和**技术实现**一些方面剖析此项目的开发过程以及采到的坑。

## 项目截图
<img src="https://user-gold-cdn.xitu.io/2020/7/1/17308b5c66a54ae5?w=286&h=500&f=gif&s=1854225" width="200">
<img src="https://user-gold-cdn.xitu.io/2020/7/1/17308b647f0cba7f?w=286&h=500&f=gif&s=838667" width="200">
<img src="https://user-gold-cdn.xitu.io/2020/7/1/17308b6220974f7e?w=286&h=500&f=gif&s=1786516" width="200">
<img src="https://user-gold-cdn.xitu.io/2020/7/1/17308b316156fd95?w=286&h=500&f=gif&s=954705" width="200">
<img src="https://user-gold-cdn.xitu.io/2020/7/1/17308b33d9cb205e?w=286&h=500&f=gif&s=1187009" width="200">

## 项目技术栈

1. 前端
    - `vue`开发项目核心框架
    - `axios` HTTP请求模块
    - `lib-flexible` 移动端屏幕适配方案
    - `better-scroll` 仿`IOS`效果的移动端滚动库
    - `normalize.css` 第三方`css`样式初始化模块
    - `es 6/7` 下一代`javascript`语法
2. 后端
    - `express` 搭建服务端应用核心框架
3. 开发
    - `vue-cli` 项目初始化脚手架
    - `vue-devtools` 项目开发环境调试工具
    - `vscode` `chrome` `git` `macbookpro`
    
4. 部署
    - `代码托管仓库` [https://github.com/konglingwen94/vue-elm-sell](https://github.com/konglingwen94/vue-elm-sell)
    - `线上地址` [123.56.124.33:5000](123.56.124.33:5000)
    
## 应用功能
- [x] 商品页
     - [x] 商品分类导航和商品列表的联动效果
     - [x] 点击商品分类菜单展示对应商品列表信息
     - [x] **添加/删除**商品到购物车
     - [x] 点击商品进入到详情页面
     - [ ] 商品添加到购物车动画效果
     - [ ] 页面滚动到对应商品类别时的标题吸顶效果
- [x] 评论页
    - [x] 综合评论信息渲染
    - [x] 切换评论筛选项按钮展示对应的信息
    - [x] 选择展示`是否有内容的评论`
- [x] 商家页
     - [x] 商家店铺信息展示
     - [x] 收藏店铺
     - [x] 商家实景图片具有`bounce`效果的滑动显示
- [x] 应用头部
    - [x] 点击展示详情
    - [ ] 公告信息动态滚动显示
- [x] 购物车
    - [x] 根据商品个数显示不同的状态
    - [x] 购物车商品列表
    - [x] 支付弹窗
    - [x] 清空购物车
    - [x] 增加/删除商品

- [ ] 应用局部优化

> `bounce效果`是指在应用中页面位置滚动到一个端点继续滑动时出现反弹的效果,常见场景是`IOS`系统应用滑动效果
## 功能难点

### 商品导航和内容的左右联动效果
#### 效果演示

> 完整的组件代码点[https://github.com/konglingwen94/vue-elm-sell/blob/master/src/views/goods/index.vue](https://github.com/konglingwen94/vue-elm-sell/blob/master/src/views/goods/index.vue)。

![](https://user-gold-cdn.xitu.io/2020/6/26/172f0246ad94e34e?w=296&h=500&f=gif&s=1465857)


#### 思路
由于商品导航和内容是两个独立的滚动容器，当滚动到一个目标内容块时怎么才能激活它所关联的导航项呢?我们知道导航项列表和内容列表在排列顺序上是一致的，如果能计算出内容滚动位置处在对应区间块的索引，也就得到了导航列表应该激活的目标索引，然后就可以用`Vue`数据驱动视图的思想去实现这一切。

 

>容器的左右联动效果是指容器滚动到目标内容时激活其关联的导航菜单项并滚动到可视区域。

#### 逻辑实现
找到要激活的目标导航项索引的第一步需要把商品内容的各个类别块在容器内的纵坐标位置存储起来（给之后找到激活的目标索引提供比较对象），由于列表内容时动态渲染的，所以这里需要等所有数据已经渲染完成后才能操作，下面直接看代码演示吧！

`template`部分
```HTML
<template>
    /* 这里只显示部分代码*/
     <ul class="foods-list">
          <li ref="foodsGroup" class="foods-group" v-for="(item,index) in data" :key="index">
            <dl class="foods-group-wrapper">
              <dt :class="{fixed:currentIndex===index}" class="foods-group-name">{{item.name}}</dt>
              <dd
                class="foods-group-item"
                v-for="(food ,key) in item.foods"
                :key="key"
              >
                 {{food.name}}
              </dd>
            </dl>
          </li>
        </ul>
</template>
```
`script`部分
```js
 
export default {
    data(){
      return {
          currentIndex: 0,//导航项激活的索引
          currentFood: {},
          data:[],
          sectionHeight: [0],//第一个高度块坐标`y`值为`0`
          // 渲染完成后的值为 `[0,1281,1459,1612,2000,2270,2565,2952,3574,4436]`
      }  
    },
    created() {
        request
          .get("/goods")
          .then(response => {
            this.data = response;
          })
          .then(() => {
            setTimeout(() => {
              const sections = this.$refs.foodsGroup;
    
              sections.reduce((prevTotal, current) => {
                const sectionHeight = prevTotal + current.clientHeight;
                this.sectionHeight.push(sectionHeight);
    
                return sectionHeight;
              }, 0);
            });
        });
  }
}

```

有了各个商品块的`y`坐标，下一步就需要注册容器元素的滚动事件了，在回调函数里通过找到实时滚动位置`disanceY`处在`sectionHeight`数组中两个相邻元素之间的位置从而就得到了待激活导航索引`currentIndex`的值，具体代码实现如下

```HTML
<template>
    <div>
        <!--导航菜单-->
         <scroll class="menu">
            <ul class="menu-list">
              <li
                @tap="selectMenu(index)"
                class="menu-item"
                :class="{selected:currentIndex===index}"
                v-for="(item,index) in data"
                :key="index"
              >
               <span>{{ item.name}}</span>
              </li>
            </ul>
          </scroll>
          
        <!--商品内容-->
        <scroll ref="foodsScroll" @scroll="onFoodScroll" class="foods">
        <!--这里省略商品内容模板的代码-->
        </scroll>
    
    </div>
</template>

```

```javascript
export default {

    // 这里省略其他代码
    
    methods:{
        onFoodScroll({ x, y }) {
          const distanceY = Math.abs(Math.round(y));
          for (let index = 0; index < this.sectionHeight.length; index++) {
            if (
              distanceY >= this.sectionHeight[index] &&
              distanceY < this.sectionHeight[index + 1]
            ) {
              this.currentIndex = index;
            }
          }
        }
    }
}
```

> 完整的组件代码点[https://github.com/konglingwen94/vue-elm-sell/blob/master/src/views/goods/index.vue](https://github.com/konglingwen94/vue-elm-sell/blob/master/src/views/goods/index.vue)。由于左右两侧的布局容器都是基于`better-scroll`实现的页面滚动，所以这里需要侦听`better-scroll`提供的`scroll`事件而不是浏览器原生的滚动事件。查看`better-scroll`的`scroll`事件`API`点[这里](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll-api.html#%E9%92%A9%E5%AD%90)。

### 添加/删除 商品到购物车

#### 效果截图

![](https://user-gold-cdn.xitu.io/2020/6/30/1730353cd1f67930?w=287&h=500&f=gif&s=1630266)

> 完整代码[https://github.com/konglingwen94/vue-elm-sell/blob/master/src/components/food-picker/index.vue](https://github.com/konglingwen94/vue-elm-sell/blob/master/src/components/food-picker/index.vue)
#### 思路

添加商品到购物车是一个多场景的功能，由于这里的购物车功能是一个多页面联动的效果，购物车商品数量的实时更改也需要同步到商品内容页和商品详情页。从功能映射到`javascript`语言数据结构层面的话，不难想到**对象引用传递**的特点可以作为实现此功能的底层架构思路，那就让我们去实现它吧。

#### 实现
为了统计商品的数量。首先需要给每一个商品信息对象添加一个默认值为`0`的`count`属性，添加后的对象长这样
```
{
  "count": 0, // 此变量用来存储添加到购物车的数量
  "name": "皮蛋瘦肉粥",
  "price": 10,
  "oldPrice": "",
  "description": "咸粥",
  "sellCount": 229,
  "rating": 100,
  "info": "一碗皮蛋瘦肉粥，总是我到粥店时的不二之选。香浓软滑，饱腹暖心，皮蛋的Q弹与瘦肉的滑嫩伴着粥香溢于满口，让人喝这样的一碗粥也觉得心满意足",
  "ratings": [
    {
      "username": "3******b",
      "rateTime": 1469261964000,
      "rateType": 1,
      "text": "",
      "avatar": "http://static.galileo.xiaojukeji.com/static/tms/default_header.png"
    }
  ],
  "icon": "http://fuss10.elemecdn.com/c/cd/c12745ed8a5171e13b427dbc39401jpeg.jpeg?imageView2/1/w/114/h/114",
  "image": "http://fuss10.elemecdn.com/c/cd/c12745ed8a5171e13b427dbc39401jpeg.jpeg?imageView2/1/w/750/h/750"
}

```
由于每一个商品项都有一个添加到购物车的**数量选择器**功能，这样我们直接给**商品数量选择器**组件设计一个名为`foodInfo`的对象类型`props`，这样在**增加/减少商品数量**的时候直接操作`foodInfo`的`count`属性来实现同步数据的效果。

#### /components/food-picker.vue 组件代码

```HTML
<template>
  <div class="food-picker" @click.stop>
    <div class="reduce-wrapper" @click="reduce">
      <i class="iconfont reduce"></i>
    </div>
    <div class="counter">{{foodInfo.count}}</div>
    <div class="add-wrapper" @click="add">
      <i class="iconfont add"></i>
    </div>
  </div>
</template>
<script>
export default {
  name: "food-picker",
  props: {
    foodInfo: {
      type: Object,
      default: () => ({})
    }
  },
  methods: {
    reduce() {
      if (parseInt(this.foodInfo.count) > 0) {
        this.foodInfo.count--;
      }
    },
    add() {
      this.foodInfo.count++;
    }
  }
};
</script>
<style lang="less" scoped>
.food-picker {
  min-width: 180px;
  max-width: 200px;

  display: flex;
  align-items: center;
  width: 100%;
  justify-content: space-between;
  .iconfont {
    color: #00a0dc;
    font-size: 38px;
  }
  .counter {
    // margin: 0 20px;
  }
}
</style>
```

## 下一个目标

基于目前已经实现的功能，整个应用的数据都是以`json`文件的格式存储在服务器，服务端并没有可以用来**增删改查**的`API`接口可供使用。下一步我计划做出**管理后台**和服务端`API`用来管理前端页面的数据，使所有模块的数据都是可配置的，这样前端所渲染出来的数据也都是动态的,能够整合三端到一个项目也满足了当下`Web`全栈开发的场景需要。


## 总结

通过真实的开发这样一个复杂交互的应用，自己对`Vue`在实际业务场景中的使用和理解有深入了一步。深入理解了`Vue`的**数据驱动视图改变**的思想，熟练的掌握了**组件化开发**项目的流程，同时也感受到所带来的便利，为自己接下来预备做的中大型项目建筑好了桥梁。

## 支持

如果本项目对您学习有帮助，请您动手点个`star`[https://github.com/konglingwen94/vue-elm-sell](https://github.com/konglingwen94/vue-elm-sell)。也希望您继续关注我的动态[https://github.com/konglingwen94](https://github.com/konglingwen94)，有了您的支持我会有动力开源更多有趣的项目。

欢迎点赞和留言，谢谢！

> 本篇文章属于作者原创，转载参考请注明出处，谢谢！


