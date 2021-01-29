
## 创建目录结构

```javascript
vue-swiper/
├── src/
│   ├── components/ //内置组件
│   │   ├── indicator.vue  // 指示器组件
│   │   └── item.vue  // 单个轮播图容器组件
│   ├── main.js  // 项目出口
│   └── main.vue  //组件出口
├── README.md  
├── package.json
└── vue.config.js  // 组件打包配置文件
```

## 项目结构解析

我们知道一个轮播图是由其容器和内容构成的，这里首先把整个轮播图组件拆分为入口组件和其要用到的子组件（指示器组件可以根据自己的项目维护方式自由拆分，这里把它单独拆出来便于以后维护）。以往我们在使用第三方组件的时候可能会看到其组件就是单独的一个入口，所有的属性传入和事件监听都是放在了一个大的组件上了，其实这种封装的方式后期是不好维护的，而且在使用的时候也是不容易发现出现的问题，为了能让组件自身更具有语义化以及开发当中便于调试和后期组件的维护，我们把容器和内容分离开拆成两个组件，这样用户在模板中书写组件时候就能像使用普通的 HTML 标签一样嵌套使用了。

## 前提

以往我们开发一个页面中的轮播图可能会牵扯到大量手动的 DOM 样式操作，这里带来的问题就是我们关心的视图变化和逻辑操作混到了一起，关注点没有分离开，无论从维护还是代码的可读性这种方式都不是最优的。现在我们是基于 Vue 开发的这个组件，这样就可以利用它最核心的思想（数据驱动视图改变）开完成这个组件的开发。

## 数据驱动动画

有了数据驱动的这个主要思想，我们就可以围绕它展开组件开发了。首先把轮播图播放动画当中能用到的状态变量进行初始化

```javascript
// main.vue
data(){
  return {
    reversing:false,//控制动画播放到首尾时无缝跳转的开关
    swiperItemCount:0,// 初始化传入的轮播图个数
    index:0 // 控制轮播图当前位置的索引
  }
},
computed:{
  scrollItemCount(){  // 内容实际存在的图片个数
    return this.swiperItemCount+2 // 组件初始化以后需要复制传进来的首尾两张图片到指定位置，所以这里需要加上2
  }
}
```

之所以要定义 reversing 这个变量是因为当动画播放到首尾端点的时候，我们要瞬间跳转到对应的首尾位置，然后就可以更改这个变量的值来关闭相应的动画以达到用户视觉上无缝滚动的效果。

至于 index 其语义大概已经描述了它所需要做的事情了，就是通过更改这个索引的值以驱动图片的位置移动，这样就有了视觉上动画的效果。相关的代码如下

```javascript
 watch: {
    index(newIndex, oldIndex) {
      const endIndex = this.scrollItemCount - 1
      if (newIndex === endIndex && newIndex > oldIndex) {
        setTimeout(() => {
          this.reversing = true
          this.index = 1
          setTimeout(() => {
            this.reversing = false
          }, 100)
        }, this.duration)
      } else if (newIndex === 0 && newIndex < oldIndex) {
        setTimeout(() => {
          this.reversing = true
          this.index = endIndex - 1
          setTimeout(() => {
            this.reversing = false
          }, 100)
        }, this.duration)
      }
    },
 }
```

这里通过观测动画播放的当前位置这个变量，我们在相应的时机更改它的值来达到整个包装容器的瞬间移动，这样也就产生了图片播放连续的动画效果了。

## 相关的技术要点

###前提

单个的图片内容是以 slot（相关 Api 查看[这里](https://cn.vuejs.org/v2/api/#vm-slots)，本片文章不做介绍）的方式接收的，我们知道当前组件的$slot 属性存储的是 vnode(不了解 vnode 的看[这里](https://cn.vuejs.org/v2/api/#VNode-接口)，同样不过多介绍)。

> 匿名插槽内容筛选

有了 slot，组件内部可以接受外界传进来的一切内容，而我们这里只需要组件定义的指定子组件，所以在组件启动后还需要对默认的匿名插槽重新处理后才可以使用，让我们看代码吧

```javascript
 created() {
    this.$slots.default = this.$slots.default.filter(vnode => {
      return (
        vnode.componentOptions && vnode.componentOptions.tag === 'swiper-item'
      )

      // swiper-item 取决于注册的指定组件名称
    })
    this.swiperItemCount = this.$slots.default.length
  },
```

了解 Vue 虚拟 DOM 渲染原理的同学应该知道，每一次的 vnode 更新都会导致页面组件的冲渲染，代码中通过过滤需要的 vnode 重新赋值到接收匿名插槽的接口上，这样 Vue 内部通过检测 vnode 的变更会渲染新的 vnode 到组件视图上，接下来就可以调整内容节点的结构了。

> 如何复制外界传入的首尾图片

vnode 有一个 tag 属性存储了它所渲染的真实 DOM 的引用，这样我们就可以通过相关的 DOM 操作 Api 来复制这些节点从而达到我们的目的，相关的代码片段在这里

```javascript
mounted() {
  const firstItem = this.$slots.default[0].elm
  const lastItem = this.$slots.default[this.$slots.default.length - 1].elm
  this.$refs.wrapper.appendChild(firstItem.cloneNode(true))
  this.$refs.wrapper.insertBefore(lastItem.cloneNode(true), firstItem)
  this.autoplay && this.play()
},
```
## 组件效果展示

![good-swiper](https://user-gold-cdn.xitu.io/2020/4/2/1713b50d29d9f6ca?w=674&h=256&f=gif&s=2166297)


## 总结

文章通过介绍 Vue 数据驱动的思想实现了一些动画效果，当我们想封装一些别的组件的时候同样可以利用这一点来达到各种各样的需求。
至于组件其他功能的具体实现过程这里就不在介绍了，有兴趣的同学可以查看本项目的 Github 仓库：[good-swiper](https://github.com/konglingwen94/good-swiper)

备注：本篇文章属于作者原创，转载请标注出处，谢谢！
