## 初衷

现如今社区上基于`Vue`的项目已经多如牛毛了，为了提升自己对`vue`的进一步理解，一直想找一个界面好看，功能完成的项目练练手。本人在逛各大招聘网站的时候发现了`字节跳动`的官方招聘网站`https://job.bytedance.com/society`很适合自己练手。我在看到这个网站后翻了翻其源代码发现这个项目并不是由当下比较流行的框架`Vue`实现的，思考过后那我能不能用`Vue`重新打造一个复刻版的网站呢?

>此项目仅参考了原版网站的界面设计和功能特点，功能实现方式和代码设计均是由本人独立构思开发完成，预览点[这里](http://123.56.124.33:3000)

## 数据从哪来？

一个完成的上线项目离不开完整的数据，那么我要做的这个项目真实数据要怎么才能拿到呢?于是自己又默默的打开了原版网站的开发者工具，在`Network`面板里发现了浏览器请求到的官方`API`接口。找到了`API`接口就省心读了，问题是像字节跳动这样的公司对服务端API请求肯定会做跨域访问权限的限制，就算某一个接口能成功的请求到数据，对于一个想要长期作为自己项目访问的接口使用来说也是不稳定的。于是我又想到了接口代理的实现方案，大概的实现思路是使用`express`搭建一个自己的服务器，包括项目上线后静态资源的托管都会用到。

## 服务端接口代理的小技巧

对于在浏览器端抓到的`API`数据接口，纯粹的分析其地址和各种各样的参数无疑是很麻烦的一件事情，有没有一种办法可以一键复用它呢?答案是肯定的!由于node端没有原生的`fetch`请求方法，这里需要借助一个第三方的node模块`node-fetch`，这个是可以直接用npm安装的类似于浏览器端原生的`fetch`请求模块。有了它我们在使用浏览器`Network`面板里面的接口一键复制功能，具体操作请看下面的演示，详细的API代码案例请点击[这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/master/server/controller/jobs.js)
![image](https://upload-images.jianshu.io/upload_images/22531718-f3b3e3937fce6e77?imageMogr2/auto-orient/strip)
```!
此功能只有高版本的chrome 浏览器有此功能，如果您的浏览器没有此复制选项，请您升级到最新的浏览器后在使用
```

## 项目技术架构

为了进一步的提高自己的技术水平和更好的加深对Vue的理解，我选择了零代码开发所有的页面功能（没有使用任何第三方UI库）。

- 项目前端技术栈
    - Vue 主框架
    - vue-router 路由跳转的官方插件
    - lodash  一个javascript的函数工具库
    - axios  负责HTTP请求的插件
- 服务端技术栈
    - express 搭建`web`服务器的`nodejs`框架
    - node-fetch 类似于浏览器端的fetch请求的polyfill
    - connect-history-api-fallback 解决单页面应用程序在`history`模式下访问服务端出现`404`的中间件
- 项目开发工具
    - vue-cli 快速搭建`Vue`工程的官方脚手架
    - less `css`预处理器
    - vscode 轻量的代码编辑器
    - postman 测试调试`API`接口的工具
    - vue-devtools `Vue`项目官方调试工具
    - chrome 应用**运行/调试**环境
    - git 开源版本控制系统
- 部署环境
    - 阿里云服务器线上地址 [http://123.56.124.33:3000](http://123.56.124.33:3000)
    - git远程仓库地址`git@github.com:konglingwen94/vue-bytedanceJob.git`
    - github 代码托管仓库[https://github.com/konglingwen94/vue-bytedanceJob](https://github.com/konglingwen94/vue-bytedanceJob)
    
## 项目源码目录

![image](https://upload-images.jianshu.io/upload_images/22531718-4bc6bac5ea35a47e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 项目重要功能剖析

### 分页器组件 components/pagination.vue 查看源代码点[这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/master/src/components/Pagination.vue)

本人在开发这个分页器组件之前也是参考了多个网站的分页功能，各种各样的分页功能各不相同，挑选之后最终确定了自己比较认可的一个，此组件实现的功能如下图所示
    
![image](https://upload-images.jianshu.io/upload_images/22531718-d18ec6ab05f80abc?imageMogr2/auto-orient/strip)
从上图可以看出这是一个功能完成的分页器组件，基础性的代码这里就不过多的介绍了，具体实现点击[这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/master/src/components/Pagination.vue)。下面就主要分析一下在开发过程中遇到的难点！


当鼠标点击分页的数字按钮时，整个分页条会做相应的动态切换。当总页数超过一定的数值后，或者页面切换到某一个范围时会出现相应的省略号代替隐藏的页码数字展示。那这个功能逻辑应该怎么实现呢?

良好的思路是写出优雅代码的第一步，我把分页可能出现的状态分为四种情况

1.  第一个省略号出现在最大页数前面时
2. 分页条出现两个省略号时
3. 省略号出现在最小页数后面时
4. 分页器总页码小于默认展示的页码个数（分页条没有出现省略号）

根据以上列出的几种情况就可以使用代码去实现了，这里我使用`Vue` 的一个计算属性`visiblePagers`进行了动态展示所有出现的页码，组件完整的代码如下

```
<template>
  <div class="pagination">
    <ul class="pagination-list">
      <li
        title="上一页"
        @click="$emit('update:currentPage',Math.max(1,currentPage-1))"
        class="pagination-item"
      >
        <span><</span>
      </li>
      <li
        class="pagination-item"
        :class="{current:currentPage===item}"
        v-for="(item,index) in visiblePages"
        @click="change(item)"
        :key="index"
      >
        <span>{{item}}</span>
      </li>
      <li
        title="下一页"
        @click="$emit('update:currentPage',Math.min(totalPage,currentPage+1))"
        class="pagination-item"
      >
        <span>></span>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  name: "Pagination",
  props: {
    total: Number,
    perPage: {
      type: Number,
      default: 10
    },
    currentPage: {
      type: Number,
      default: 1
    },
    pagerCount: {
      type: Number,
      default: 9
    }
  },
  computed: {
    totalPage() {
      return Math.ceil(parseInt(this.total) / this.perPage);
    },
    visiblePages() {
      let pages = [];
      const currentPage = Math.max(
        1,
        Math.min(this.currentPage, this.totalPage)
      );
       

      if (this.totalPage <= this.pagerCount) {
        for (let i = 1; i <= this.totalPage; i++) {
          pages.push(i);
        }
        return pages;
      }

      if (currentPage >= this.totalPage - 3) {
        pages.push(1, "...");
        const minPage = Math.min(currentPage - 2, this.totalPage - 4);
        for (let i = minPage, len = this.totalPage; i <= len; i++) {
          pages.push(i);
        }
      } else if (currentPage <= 4) {
        const maxPage = Math.min(Math.max(currentPage + 2, 5), this.totalPage);
        for (let i = 1; i <= maxPage; i++) {
          pages.push(i);
        }
        pages.push("...", this.totalPage);
      } else {
        pages.push(1, "...");
        for (let i = currentPage - 2; i <= currentPage + 2; i++) {
          pages.push(i);
        }
        pages.push("...", this.totalPage);
      }
      return pages;
    }
  },
  methods: {
    change(num) {
      if (typeof num !== "number") {
        return;
      }
      this.$emit("current-change", num);
      this.$emit("update:currentPage", num);
    }
  }
};
</script>
<style lang="less" scoped>
.pagination-list {
  display: flex;
}
.pagination-item {
  margin-right: 4px;
  cursor: pointer;
  padding: 8px;
  &:hover {
    color: @main-color;
  }
  &.current {
    color: @main-color;
  }
}
</style>

```

### 复选穿梭框组件 components/checkbox-transfer.vue 源代码[地址](https://github.com/konglingwen94/vue-bytedanceJob/blob/master/src/components/Checkbox-Transfer.vue)

 
#### 效果图


![image](https://upload-images.jianshu.io/upload_images/22531718-496411879c2b1c5c?imageMogr2/auto-orient/strip)

#### 实现过程

对于这个组件功能的开发，我真的是煞费苦心，一言难尽。首先市面上没有一样的功能需求用例可供参考，其次在开发的过程中组件各种逻辑的实现也是在摸索着进行实现。在花费了一定时间仍没有较好的思路后，我默默的打开了**世界上最大的程序员同性交友平台 github**一番搜索，最终在开源项目基于Vue的组件库`element-ui`中的`transfer`组件中找到了可参考实现的逻辑实现方式。

#### 重点逻辑分析

`template` 部分代码

```
<template>
  <div class="checkbox">
    <h2>{{title}}</h2>

    <ul class="checkbox-list">
      <li class="checkbox-item" v-for="(item, index) in targetData" :key="index">
        <input
          @change="check(item, $event)"
          type="checkbox"
          :id="item[props.key]"
          :checked="checked[index]"
        />
        <label :for="item[props.key]" class="label-text">{{ item[props.label] }}</label>
      </li>
    </ul>
    <div class="search" v-if="sourceData.length">
      <input
        @blur="onInputBlur"
        @focus="focusing = true"
        class="search-input"
        :class="{focusing}"
        :placeholder="placeholder"
        type="text"
        v-model="filterKeyword"
      />
      <ul class="search-list" v-show="focusing">
        <li
          v-for="item in filterableData"
          :key="item[props.key]"
          class="search-item"
          @click="addToTarget(item)"
        >
          <span>{{ item[props.label] }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

```

对于这样一个交互复杂的复选穿梭框而言，定义好其基本的初始化数据状态是第一步要做的。对于此组件调用的时候，模板会传入一个属性名为`data`（这里是一个数组）的`props`选项作为组件的数据来源，接下来要关注的焦点就转移到了组件内部数据交互的各种实现方式上了。我首先在组件状态`data`里面定义了一个名叫`targets`的数组类型的变量用来存储默认展开的复选框列表项`key`值，然后根据`data`和`targets`这两个基础数据状态又可以派生出两个计算属性`sourceData`和`targetData`用来分别渲染展开和隐藏起来的复选框选择项。至此组件基本的静态模板渲染的逻辑就构思完成了。相关部分代码如下


组件`script`部分
```
<script>
export default {
  name: "checkbox-transfer",
  data() {
    return {
      focusing: false,
      filterKeyword: "",
      targets: []
    };
  },
  props: {
    data: {
      type: Array,
      default: () => []
    },
  },
  computed: {
    targetData() {
      return this.targets
        .map(key => {
          return this.data.find(item => item[this.props.key] === key);
        })
        .filter(item => item && item[this.props.key]);
    },
    sourceData() {
      return this.data.filter(
        item => this.targets.indexOf(item[this.props.key]) === -1
      );
    },
  },
```

另外此组件也拥有`Vue`组件代表性的双向数据绑定的特点，使用`v-model`的指令可以默认指定复选框选中的选项，有关这一块的逻辑实现在这里就不在赘述了，相关逻辑代码看下面

组件`script`部分
```
<script>
export default {
  name: "checkbox-transfer",
  props: {
    value: {
      type: Array,
      default: () => []
    }
  },

  computed: {
    checked() {
      return this.targets.map(key => this.value.includes(key));
    },
  },
 methods:{
     check(item, e) {
      if (!e.target.checked) {
        const delIndex = this.value.indexOf(item[this.props.key]);
        if (delIndex > -1) {
          this.value.splice(delIndex, 1);
        }
      } else {
        if (!this.value.includes(item[this.props.key])) {
          this.value.push(item[this.props.key]);
        }
      }
    
      this.$emit("check", e.target.checked, item[this.props.key]);
      this.$emit("input", this.value);
    }
  }
 }
   
```



### 首页头部导航栏交互显示功能 
查看源码点[这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/master/src/views/Home.vue)

#### 效果

![image](https://upload-images.jianshu.io/upload_images/22531718-bf18b947dcccf614?imageMogr2/auto-orient/strip)

#### 功能介绍

1. 顶部的导航栏在页面向下滚动的过程中有一个吸附顶部的功能
2. `banner`视频部分滚动出页面可视区域后导航栏更换主题颜色
3. 导航栏根据页面滚动方向的不同隐藏和显示。

#### 实现思路

导航栏功能的前两个实现起来比较简单，这里就不多介绍了。下面主要分析一下功能中的第三点。

根据页面滚动的方向判断导航栏显示状态要怎么实现呢？单纯的做一个显示状态的切换对于熟练使用`Vue`的同学来说再简单不过了，这里的坑就在于如何判断页面在滚动过程中的滚动方向呢？说道这里，有人一定能想到可以使用浏览器原生`API`监听元素滚动的事件，在事件`scroll`的回调函数中进一步处理逻辑判断。能想到这一点对于我们要实现的最终目标有迈进了一大步，那么浏览器`HTML`元素的`scroll`事件能提供给我们使用的回调参数是有限的，就是说这个事件对象没有直接提供此次滚动的方向信息。所以这个问题就需要我们手动去解决了。

#### 手动封装监听元素滚动的函数

为了判断出元素此次滚动事件相对于上次滚动时的方向，我们需要记录上一次的滚动事件信息并存储起来，然后通过比对两次滚动事件的坐标值判断出此次页面滚动的方向（这里只做滚动向下或者向上的判断）。为了让这一个判断元素滚动方向的逻辑有更好的复用性，我把它单独抽离成了一个工具函数，当我们需要用到这个逻辑时就可以直接拿来复用，具体实现的代码如下

`helper/untilities.js`
```js
export const watchScrollDirection = function(scrollElement, callback) {
  const scrollPos = { x: 0, y: 0 };
  const scrollDirection = {
    directionX: 1,
    directionY: 1,
  };

  function onScroll(e) {
    const scrollTop = scrollElement.scrollTop || scrollElement.pageYOffset;
    const scrollLeft = scrollElement.scrollLeft || scrollElement.pageXOffset;

    if (scrollPos.y > scrollTop) {
      scrollDirection.directionY = -1;
    } else {
      scrollDirection.directionY = 1;
    }
    if (scrollPos.x > scrollLeft) {
      scrollDirection.directionX = -1;
    } else {
      scrollDirection.directionX = 1;
    }
    callback.call(scrollElement, scrollDirection,scrollPos);

    scrollPos.x = scrollLeft;
    scrollPos.y = scrollTop;
  }
  scrollElement.addEventListener("scroll", onScroll);
  return function() {
    scrollElement.removeEventListener("scroll", onScroll);
  };
};
```
页面代码实现如下

`views/home.vue`组件`script`部分
```js
import { watchScrollDirection } from "@/helper/utilities.js";

export default {
    mounted() {
        const rootVm = this.$root;
        rootVm.$emit(
          "home-scrolling",
          { directionX: 1, directionY: -1 },
          { x: document.body.scrollLeft, y: document.body.scrollTop }
        );
        this.unwatch = watchScrollDirection(window, function(...args) {
          rootVm.$emit("home-scrolling", ...args);
        });
      },
    destroyed() {
       this.unwatch();
    }
}
```
 
## 应用截图

### 首页

![image](https://upload-images.jianshu.io/upload_images/22531718-924e53098a0ebf78?imageMogr2/auto-orient/strip)

### 职位


![image](https://upload-images.jianshu.io/upload_images/22531718-b28f266aab4cb16b?imageMogr2/auto-orient/strip)

### 产品与服务


![image](https://upload-images.jianshu.io/upload_images/22531718-fc1a35f9829e831c?imageMogr2/auto-orient/strip)

### 职位详情


![image](https://upload-images.jianshu.io/upload_images/22531718-2f1f27b44f701fe1?imageMogr2/auto-orient/strip)

### 员工故事


![image](https://upload-images.jianshu.io/upload_images/22531718-b8e66dfb7026f1e1?imageMogr2/auto-orient/strip)

## 支持

阅读完本篇文章如果对您有帮助，请您点赞，谢谢！

如果想讨论项目有关问题或者参与共建，欢迎留言！

对本项目如果您有更好的**建议**或发现`bug`,欢迎提 [issue](https://github.com/konglingwen94/vue-bytedanceJob/issues)

应用线上地址: [http://123.56.124.33:3000/](http://123.56.124.33:3000/)

项目仓库地址: [https://github.com/konglingwen94/vue-bytedanceJob](https://github.com/konglingwen94/vue-bytedanceJob),欢迎`star`和`follow`,谢谢！

>本篇文章属于个人原创，转载借鉴请注明出处，谢谢！