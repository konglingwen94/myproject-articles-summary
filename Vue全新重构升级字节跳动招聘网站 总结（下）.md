 

项目仓库地址：<https://github.com/konglingwen94/vue-bytedanceJob>

项目线上预览：<http://123.56.124.33:3000>

上篇文章：[Vue全栈技术重构字节跳动招聘网站（上）
](https://juejin.im/post/6844904199289831432)

文章作者：孔令文。




## 前言

全新重构升级的项目[vue-bytedanceJob 2.0](https://github.com/konglingwen94/vue-bytedanceJob)终于完成了，特此写一篇文章总结一下，也是文章[Vue全栈技术重构字节跳动招聘网站(上)
](https://juejin.im/post/6844904199289831432)的续篇。本篇文章会重点介绍网站新版本添加的主要功能，并会对有意思的地方详细分析，还有就是对于整个项目的重新概括（技术栈）以及项目中一些典型业务逻辑的实现剖析。同时，`2.0`版本的项目还增加了几个`API`类型的自定义组件，比如，`进度条弹窗`，`消息通知`，`数据加载loading`等组件，这些组件的功能特点和开发过程我也会进行解析。所以下面的介绍是干货满满，看完了别忘记点赞留言额！

## 新增加的需求
已实现
- [x]   [用户通过邮箱登录账号](http://123.56.124.33:3000/user)
- [x] [选择本地简历文件上传](http://123.56.124.33:3000/resume/edit)
- [x] [解析本地上传的简历文件](http://123.56.124.33:3000/resume/edit)
- [x] [保存新编写的简历](http://123.56.124.33:3000/resume/edit)
- [x] [预览最新的简历](http://123.56.124.33:3000/resume)
- [x] [简历文件下载到本地](http://123.56.124.33:3000/resume)
- [x] 各个页面数据加载`loading`提示
- [x] [职位搜索框的动态吸顶](http://123.56.124.33:3000/jobs)
- [x] [简历`保存取消`操作栏动态吸底](http://123.56.124.33:3000/resume/edit)
- [x] 添加部分页面切换时的动画效果
- [x] 项目整体的优化

未实现

- [ ] 邮箱注册账号
- [ ] 手机号登录
- [ ] 简历投递 
- [ ] 项目进一步的优化

### 部分屏幕截图

简历预览页面
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e17131fe231f418dbcc8def8b2aa3f1d~tplv-k3u1fbpfcp-zoom-1.image)

简历编辑页面
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/518ee2efa1c4480eaabb9e02f6bf066f~tplv-k3u1fbpfcp-zoom-1.image)


除了以上列出来的需求已全部实现之外，针对于一些功能的重构也有很多。包括几个可以使用`API`方式调用的组件也是通过自己手动开发封装实现的，并且替换了在上一个版本中使用第三方组件库引入的组件。项目最后的展示效果实际测试下来，在业务的整体结合和组件基础功能的定制性上有了很大的改善，不仅是沉余的代码变少了，而且在业务逻辑上也更吻合了。

有了很多改进的地方，也有不足的地方，相比于第三方成熟的类库在测试用例上不够全面，部分地方还需要进一步的测试改进。其次如果你需要引入到自己的项目中使用这些组件是需要进行二次开发的，因为这些都是业务组件而不是通用组件。如果你在使用的过程中有问题或者好的建议，欢迎提[issues](https://github.com/konglingwen94/vue-bytedanceJob/issues)!

## 新增加的三个`API`组件
以下的组件均支持API的方式调用，本项目各组件的调用方法挂载到了`Vue`实例的原型对象上，根据具体用例你也可以单独引入到特定文件内使用 ，各组件详细介绍如下
### 数据加载组件`Loading`
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67dfb5307cb548e6b7735f502e74fdfb~tplv-k3u1fbpfcp-zoom-1.image)

`Loading`组件基本的使用方式同大多数类库是一样的，我也是参考了它们的`API`设计方式和参数传递方式实现的。图上显示的`loading`有两种调用状态，一个是局部调用（通过指令的方式），还有就是方法调用，这里以简化版代码做一个演示。

统一说明：代码演示中`this`均为`Vue`实例

`API`方式使用
```js
/*
@params `position`是全屏`loading`具体的定位信息，默认为`0`，可以根据需求适当调整其位置
*/ 

const  loading = this.$loading({ position: { top: 0, left: 0, right: 0, bottom: 0 } });

// 在适当的实际关闭
loading.close()
```
以指令的方式调用

```html
<template>
  <div v-loading.scrollFixed="true"></div>
</template>

```
指令的绑定值可以用一个组件状态变量代替，以便灵活控制`loading`状态，这里以简化版代码代替
`scrollFixed`是支持传递的指令修饰符，其作用是当被绑定`loading`的`DOM元素`滚动到页面视口之外时，`loading`图标永远居中显示，小伙伴们不用考虑元素滚动带来`loading`一起滚动并消失在视口内的烦恼了，想想是不是很酸爽呢！

查看组件代码点[这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/2.0/src/components/Loading/Loading.vue)

### 进度条弹窗组件`popup-progress`
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e281d84564a498392ec9033bc709751~tplv-k3u1fbpfcp-zoom-1.image)

`popup-progress`组件可以认为是`弹出式进度条`组件，由于没有想到更好的名字暂且先这样命名吧。使用过一些类库组件的同学很容易能想到他就是结合了弹窗`alert`和进度条`progress`设计的。由于属于是`全屏组件`,这里又引出了`js`众多设计模式之一的`单例模式`的概念。这个组件设计的时候也是运用了此模式，跟大多数弹框类的组件一样也算是一个比较经典的实现方式吧。那么它跟其他模态框组件不同的一点是什么呢?

由于弹框中的滚动条是一个不断变化的状态，调用此组件的时候跟别的返回`Promise`的一些组件就不一样了，比如`alert`,`confirm`,`prompt`这类组件等等。这些组件之所以可以返回`promise`是组件挂载后需要等待用户操作状态，不经过用户手动操作状态组件自身的状态是固定不变的，这跟`js`中的`promise`颇为相像。所以组件调用返回值为`promise`也就不足为奇了。但是本文中的`popup-progress`就不一样了，为了符合业务需要，组件在挂载之后，进度条的进度是一个需要变化的值。显然通过向组件传递固定状态值属性的方式就不行了，所以组件创建后返回实例自身更合适。这样的处理在组件被调用的上线文环境中返回了组件实例自身，而不是通过组件初始化的时候传递`函数`等复杂的数据类型参数以便在组件内部处理。这种设计方式让组件注册监听自定义事件也颇为方便，不仅可以链式的调用`API`，而且我个人认为这比向组件传递`监听器函数的参数`的方式更优雅。说了这么多，唯有代码演示才更直观。

```javascript
// 创建进度条弹窗
const popPro = this.$popupProgress();

//在具体的使用用例中更新`popPro`的进度值， 这里用`setInterval`模拟演示

const timer = setInterval(() => {
  //控制进度条的进度的值这里为`value`，值需要是`1-100`之间的数
  popPro.value += 10;
}, 300);

// 监听上传终止事件（点击`取消`按钮后触发）
popPro.$on("abort", () => {
  //终止上传过程后的操作， 关闭弹窗进度条
  	popPro.close()
});
//当进度条进度为`100%`的装态时触发`success`事件
popPro.$on("success", () => {
  clearInterval(timer);
  // 关闭弹窗进度条
	popPro.close()
});

```

上面注册事件的代码可以简写为

```js
this.$popupProgress()
  .$on("abort", () => {
     
  })
  .$on("success", () => {
    
  });
```

至此有关`popup-progress`组件的全部分析就到这了，如果您有好的建议或问题欢迎提[issue](https://github.com/konglingwen94/vue-bytedanceJob/issues)


>查看代码点 <https://github.com/konglingwen94/vue-bytedanceJob/tree/2.0/src/components/popup-progress>


### 消息通知组件`message`

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7504f58553f48eda129038e99d21624~tplv-k3u1fbpfcp-zoom-1.image)

#### 引言

`message`组件是日常业务开发中使用比较多的，可能追求快捷开发的同学直接就引入通用的类库了（我之前就是这么做的）。通用类库的组件有一个特点，它之所以通用是因为在功能设计上考虑了很多的业务场景和用例，自然包含的功能就很丰富。但是对于需要长期维护更新的项目来说就会带来很多问题，首先通用组件的`UI`风格和自身业务的需求并不恰好一致，其次代码量会有沉余，这些都是不可规避的问题。所以当我们在遇到类似的业务需求时，自己动手实现往往是更好的选择，这样不仅会使项目整体的代码更聚合，与此同时我们自身的技术水平也会有很大提高。

#### 基本介绍

`message`组件有三种消息状态分别为`成功`、`失败`、`警告`，对于一般复杂度的项目应该够用了。**麻雀虽小，五脏俱全**，消息通知类组件应该有的功能本组件都有，虽不如通过的轮子可传递的参数多样。仅以本项目的用例说的话足够用了，有兴趣的同学可以基于此二次开发。欢迎提`PR`和[issue](https://github.com/konglingwen94/vue-bytedanceJob/issues)。组件`API`的使用方式也保持了通用的设计方式。具体演示如下

```javacript
// 这里以成功的消息状态举例，其他两种同理
this.$message({message:"成功",type:"success"})

// 支持使用消息类型的别名直接调用
this.$message.success('成功')

// 支持传递消息关闭的时间参数，默认为`3000ms`

this.$message({message:"成功",duration:2000})
```

`message`组件和上面介绍的`popup-progress`组件相反，`API`底层的设计模式是工厂模式。所有已创建的组件实例在销毁前都存放到一个队列中。像代码中创建组件的方法每调用一次都会在视图上添加一个新消息。显示效果就如图上那样依次排列。所以在使用的时候可以多次的调用。

>查看代码点 <https://github.com/konglingwen94/vue-bytedanceJob/tree/2.0/src/components/message>

## 递归思想在业务中的运用

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac301b2fc09244b7810284d28a8ba254~tplv-k3u1fbpfcp-zoom-1.image)
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fb9ce3e1bf540b8832afa641a7c44c2~tplv-k3u1fbpfcp-zoom-1.image)

看到这两个典型用例的演示，好像跟标题中要说的`递归`没有联系，其实这里面的一个函数的实现方式正是用了`递归`的思想。回到图片中的功能本身，页面在滚动过程中某一部分动态吸底或吸顶并不复杂，这里要说的是其中的一个细节实现。

我们知道`DOM`元素的一个属性值`offsetTop`。它代表的是相对最近的定位父级元素顶部的距离偏移，那么这里又印出来了另外一个属性`offsetParent`,它代表的就是元素最近的父级定位元素。有了这两个属性，我们可以轻松的获取元素各个方向距离父元素的偏移距离。那如果我们要获取元素距离页面最顶部的距离应该怎么拿到？如果元素的`offsetParent`属性是根元素`html`的话，直接获取元素的`offsetTop`属性就行了。对于有多个父级定位元素的话，我们就需要手动逐级的查找元素的`offsetParent`元素的`offsetTop`进行累加计算。这种方式最终也可以正确的获取到结果。问题是在页面布局复杂的场景下这个方法就不好使了。如果要计算的元素层级嵌套很深，父级定位元素又多的话计算量是非常大而且容易出错。我把逐级向上查找父级元素的`offsetTop`值看成是一个手动的递归过程，既然可以用程序解决的问题，最终都将用程序来解决。直接放代码出来吧

```js
function getOffsetTop(node, targetOffsetParentNode, offsetTopSum = 0) {
  offsetTopSum += node.offsetTop;

  if (node.offsetParent !== targetOffsetParentNode) {
    return getOffsetTop(node.offsetParent, targetOffsetParentNode, offsetTopSum);
  }
  return offsetTopSum;
}

//用例：获取元素`$0`距离根元素顶部的距离`offsetTop`

const offsetTop = getOffsetTop($0,document.documentElement)
```
有了这个相对于根元素的顶部距离跟实现图中的效果有什么关联？通过比较这个值和滚动容器的滚动坐标就可以实现滚动切换定位元素的功能了，这里就不具体分析了。

功能实现源代码

>职位搜索栏滚动吸顶 [这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/2.0/src/views/Jobs.vue#L122)

>简历编辑操作栏动态吸底 <https://github.com/konglingwen94/vue-bytedanceJob/blob/2.0/src/views/ResumeEditor.vue#L974>

>查看本项目更多工具函数点 [这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/2.0/src/helper/utilities.js)



## 服务端接口抓取

关于服务端接口是如何抓取的，这里我也简单谈一下项目整个后端接口代理的整个过程。从最开始在浏览器开发者工具提供的`Network`面板里面逐个查找，接着就是用`Postman`一个一个的实际调试。说到这我想说一下自己爬数据接口的一点心得，用工具调试接口和用代码模拟调试真的是两回事。由于运行环境的不一致，往往会出现一些意料之外的问题。这里我就拿`登录`类强依赖`接口状态`的接口请求举例子，这类的接口由于需要身份验证需要很多私密字段携带到请求头里面发送到服务端，有一些请求头字段是浏览器默认不允许携带的，比如`referer`,`origin`等字段，所以需要携带这类信息发送请求的接口在浏览器是无法实现的，所以我们往往需要一个代理服务器去转发这些接口，其实本项目也是这么做的，而且代理转发请求的方式是用一个第三方中间件实现的，有兴趣的同学可以点[这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/2.0/server/app.js)查看具体实现方式。

除了上面说的这些，让我采坑比较深的就是`token`验证类型的请求接口了，且本项目就是靠验证`token`的方式实现的，有兴趣查看具体实现过程的同学在[这里](https://github.com/konglingwen94/vue-bytedanceJob/blob/2.0/src/helper/requestWithToken.js)

## 持续集成

对于一个完整上线的项目是离不开`持续集成`这道步骤的。本项目使用的是一个简化版的持续集成配置，首先上线的服务器是购买某电商的，项目自身用到持续部署插件借鉴了知名开源项目`Vue`的使用方式`yorkie`。`package.json`文件的配置为

```json
{
  "gitHooks": {
    "pre-push": "sh deploy.sh"
  }
}

```
前提是需要编写一个部署脚本`deploy.sh`，当项目每次提交到远程仓库的时候就会触发`gitHooks`配置好的脚本命令，然后就静静等待程序自动部署就ok了。

脚本部署文件内容可以按如下配置,需要添加更多步骤的话直接编写即可。

```bash
!/usr/

npm run build  # 需要构建到`server`目录

scp -r server user@192.168.1.1:/var/www/projectname

```

项目构建发布后，还需要登录云服务器开启服务，本项目使用的是`pm2`，在终端运行以下命令开启服务

```bash

cd /var/www/projectname 

pm2 start app.js
```

## 总结

通过全新重构升级的基于`Vue`的单页面应用，自己除了对`Vue`的理解和认识提高了一个层次之外，对于项目重构，升级这一系列要做的工作又熟练了一遍。从开始的`API`接口分析爬取，接着是对需求的整理分析，再到代码的最终编写完成，最后就是一遍又一遍的测试和持续集成。从而我对项目的开发步骤流程也有了更好的梳理和把握，同时自己也有很多不足的地方需要持续的补充和学习。

另外也希望对看过本篇文章的你在学习上有所帮助，共勉一下，一起加油哇！

## 支持

如果您读完文章感觉还不错的话欢迎点赞，谢谢！

如果您有好的想法和建议欢迎在评论区留言！

文章中有错误的地方欢迎指出，谢谢！

## 查看

项目仓库地址：<https://github.com/konglingwen94/vue-bytedanceJob>

项目线上预览：<http://123.56.124.33:3000>

文章作者：[孔令文](https://github.com/konglingwen94/project-articles-summary)。

