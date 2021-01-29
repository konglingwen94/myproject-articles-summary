

## 组件功能

大部分文章展示类的网站都有用户评论的功能，看了这么多的评论消息的你是不是也想封装一个通过的组件呢，以备用到的时候可以直接拿来复用。现在我们就简单概括一下这个组件的主要功能，

既然是评论消息组件，首先把用户已经评论过的消息展示出来是这个组件的基本功能，然后针对每一条消息还可以对他进行回复，这就好比现实世界中人们的交流一样，每个人发表了一段话，听到的人都可以进行回复，然后针对回复我们还可以进行回复，其实这是一个无限循环的交流过程，把这种现实中的场景放到程序中有一个专业的算法术语与之对应，那就是递归（recursive），不过现在的网站评论功能都只是展示两层的对话内容（评论和回复），今天我们要实现的是可以无限级的进行消息回复的一个功能组件。这个不仅是利用了递归的思想开发一个组件，而且当你遇到别的业务场景的时候一样可以复用这一套逻辑，比如要实现一个文件树的功能，其利用的也是数据递归的思想。那现在就让我们去封装这个组件吧!

## 创建目录结构

```javascript

├── src/ 
│   ├── components/ //内部组建
│   │   ├── message-editor.vue //评论编辑器
│   │   ├── message-group.vue //评论列表
│   │   └── message-item.vue  //单个评论
│   ├── main.js  //导出文件
│   ├── main.vue  //入口组建
│   └── util.js
├── test/
├── README.md
├── babel.config.js
├── package.json
└── vue.config.js
```



先创建好项目目录，然后把需要用到的内部组件也创建好，这样整个项目的骨架结构就搭建好了，接下来我们就开始功能的开发吧！

## 组件拆分
为了更好的实现组件模块化管理，这里把需要用到的功能组件都放到`component`的目录里面，现在就介绍一下这些组件最终都实现了怎样的功能。
```HTML
<!-- message-group.vue -->

<template>
  <div class="message-group">
    <ul>
      <li v-for="(item,index) in dataList" :key="index">
        <message-item :data="item"></message-item>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  name: "MessageGroup",
  props: {
    dataList: {
      type: Array,
      default: []
    }
  }
};
</script>

<style>
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
</style>

```
这个`message-group`组件其实就是对`message-item.vue（具体实现方式后面会详细介绍）`组件的一个列表循环展示，他就是一个消息树容器组件，这里为了开发和展示的过程中让项目组件结构的语义化更强就把他单独抽离出来了，而且当我们在浏览器`vue-devtool（vue项目开发中调试插件）`的选项面板中也能更清楚的看到每个组件所对应的视图是哪一块，这对于更好的组织组件结构是有好处的。下面用一个图片演示一下效果。

![message-tree](https://user-gold-cdn.xitu.io/2020/4/12/1716ce5af9a1ec08?w=1054&h=668&f=gif&s=6988398)

## Vue组件递归调用

组件是可以在它们自己的模板中调用自身的。不过它们只能通过 name 选项来做这件事
```javascript
name: 'unique-name-of-my-component'

```
当你使用 Vue.component 全局注册一个组件时，这个全局的 ID 会自动设置为该组件的 name 选项。

```javascript
Vue.component('unique-name-of-my-component', {
  // ...
})
```
稍有不慎，递归组件就可能导致无限循环：

```javascript
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'

```
类似上述的组件将会导致“max stack size exceeded”错误，所以请确保递归调用是条件性的 (例如使用一个最终会得到 false 的 v-if)。我们需要定义一个什么样的条件呢？如果给每一个被自身调用的递归组件打上一个层级深度的标志位（level），然后通过判断这个`level`的值来控制组件调用是不是就可以了。现在我们重写改写一下代码

```HTML
<!-- `my-component.vue` -->

<template>
  <div>
    <!-- my-component 需要全局注册，这里省略注册的代码 -->

    <my-component v-if="level===renderLevel"></my-component>
  </div>
</template>

<script>
export default {
  name:'my-component',
  inject:{
    level:{
      default:1
    }
  },
  provide(){
    return {
      level:this.level+1
    }
  },
  data(){
    return {
      renderLevel:1
    }
  }

}
</script>


```
我们先是通过Vue提供的组件选项`provide`和`inject`这两个配置项给每一个递归被调用的组件标注了一个层级`level`,然后在组件内部定义一个`renderLevel`的状态，通过判断需要展示的组件层级是否满足条件来终止递归组件的调用.

>有关`provide`和`inject`组合项的详细介绍请查看[官网](https://cn.vuejs.org/v2/api/#provide-inject)



## 留言消息组件的实现

从上面图片的最终演示效果可以看到，父留言里面会包含若干个回复评论，他们的样式和功能几乎是一样的，所以我们可以把它抽离成一个组件，这个组件又会递归的调用其自身，最终就有了一个消息树的效果。下面就把完整的代码展示出来看看具体是怎么实现的

```HTML
<!-- message-item.vue -->
<template>
  <dl>
    <dt>
      <div class="avatar-wrapper">
        <el-avatar :src="data.avatar"></el-avatar>
      </div>
      <div class="message-wrapper">
        <div class>
          <span class="nickname">{{data.nickname}}</span>&nbsp;&nbsp;&nbsp;
          <time>{{new Date(data.createdAt) | dateFormat}}</time>
        </div>

        <p class="content">
          {{data.content}}
          <span
            v-if="data.replyToUser"
          >//@{{data.replyToUser.nickname}}:{{data.replyToUser.content}}</span>
        </p>
        <div class="footer-action">
          <div class="message-statis">
            <el-button type="text" @click="replyHandler">回复</el-button>
            <el-button
              v-if="data.children && data.children.length"
              type="text"
              @click="toggleExpandPanel"
            >
              {{isExpanded?`收起回复`:`${replyCount}条回复`}}
              <i
                :class="!isExpanded?'el-icon-arrow-down':'el-icon-arrow-up'"
              ></i>
            </el-button>
          </div>
          <div class="append-right">
            <span class="thumb-button">
              {{data.thumbupCount}}
              <i class="el-icon-thumb" @click="thumbClicked(data)"></i>
            </span>
          </div>
        </div>
        <div class="editor-container" ref="editorContainer"></div>
      </div>
    </dt>
    <!-- <el-divider v-if="level===1"></el-divider> -->
    <el-collapse-transition>
      <dd class="reply-container" v-show="isExpanded" v-if="replyCount" ref="messageTreeContainer">
        <message-group :dataList="data.children"></message-group>
        <!-- <div class="loading-more" @click="loadMore" v-if="replyCount>=1">查看更多</div> -->
      </dd>
    </el-collapse-transition>
  </dl>
</template>
<script>
// import _ from 'lodash'
import { dateFormat } from '../util'

export default {
  name: 'MessageItem',

  inject: {
    level: {
      default: 1
    },
    $editor: '$editor',
    $messageTree: '$messageTree'
  },

  provide() {
    return {
      level: this.level + 1
    }
  },
  data() {
    return {
      isExpanded: this.$messageTree.expandLayer > this.level,
      hasEditor: false
    }
  },
  mounted() {
    const self = this
    this.$refs.editorContainer.addEventListener('DOMNodeInserted', function(e) {
      if (e.target === self.$editor && e.relatedNode === this) {
        self.hasEditor = true
      }
    })
    this.$refs.editorContainer.addEventListener('DOMNodeRemoved', function(e) {
      if (e.target === self.$editor && e.relatedNode === this) {
        self.hasEditor = false
      }
    })
  },
  filters: {
    dateFormat
  },
  watch: {
    '$messageTree.expandLayer': function(value) {
      this.isExpanded = value > this.level
    },
    isExpanded(value) {
      this.$messageTree.$emit('tree-expanded', this.data, value)

      if (!value) {
        if (!this.$refs.messageTreeContainer.contains(this.$editor)) {
          return
        }
        this.$editor.remove()
      }
    }
  },
  props: {
    data: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    replyCount() {
      return this.data.children && this.data.children.length
    }
  },

  methods: {
    loadMore() {
      const payload = { ...this.data }
      delete payload.children
      this.$emit('load-more', payload)
    },
    thumbClicked(item) {
      this.$messageTree.$emit('on-thumbup', item)
    },
    replyHandler() {
      if (!this.$refs.editorContainer.contains(this.$editor)) {
        this.$messageTree.showEditor()
        this.$refs.editorContainer.appendChild(this.$editor)
      }

      if (this.$messageTree.editorType === 'default') {
        this.$nextTick(() => {
          // this.$messageTree.$refs.textarea.focus()
        })
      } else {
        const payload = {
          ...this.data
        }

        delete payload.children
      this.$messageTree.$emit('on-reply', payload)
      }
    },
    toggleExpandPanel() {
      this.isExpanded = !this.isExpanded
    }
  }
}
</script>
<style lang="less" scoped>
@duration: 300ms;

dt {
  display: flex;
  .avatar-wrapper {
    margin-right: 8px;
  }
  .message-wrapper {
    width: 100%;
    font-size: 14px;

    time {
      color: #909399;
      vertical-align: middle;
    }
    .footer-action {
      display: flex;
      justify-content: space-between;
      align-items: center;
      .append-right {
        .thumb-button {
          color: #409eff;
          cursor: pointer;
        }
      }
    }
  }
}
dd.reply-container {
  background: #fafbfc;
  padding: 20px;
  margin-top: 20px;
}
.loading-more {
  user-select: none;
  text-align: center;
  font-size: 14px;
  color: #444;
  cursor: pointer;
}

.message-reply-enter,
.message-reply-leave-to {
  transform: translateX(-100%);
}
.message-reply-active {
  transition: all @duration;
}
.message-reply-move {
  transition: all @duration;
}
.message-reply-item {
  transition: all @duration;
}
</style>
```
## 最终效果

![message-tree](https://user-gold-cdn.xitu.io/2020/4/12/1716ce5af2228fb2?w=744&h=709&f=gif&s=173891)
![message-tree](https://user-gold-cdn.xitu.io/2020/4/12/1716ce5aba692b16?w=744&h=709&f=gif&s=94392)
　
## 总结
通过对留言消息树组件的封装，我们对编程算法中递归的使用有了更深入的了解。递归不仅在数据处理时会用到，在展示视图的场景下依然有很大的用处，这样组件的递归展现视图的方式会让我们的代码更干净，更便于维护，同时还有更好的逻辑复用能力。文章的最后附上本项目的[Github地址](https://github.com/konglingwen94/message-tree)

>备注：本片博文属于作者原创，转载请注明出处，谢谢!
