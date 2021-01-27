 
## 背景

在开发完成商家店铺的[客户端](https://github.com/konglingwen94/vue-elm-seller)后，自己掌握了客户端一些开发技术，并且也从中积累到了项目经验。由于客户端的商家应用页面是基于静态数据渲染的，所以在真实的应用实践中，静态的页面数据并不符合实际的应用场景需要。如果我们要定期的更换页面上的内容的话，对于普通的应用使用者来说是不容易的。可能用到最笨拙的办法就是联系应用的开发人员修改原项目内容才能实现，但是要想一劳永逸的把这个使用需求交给普通用户使用的话，使用一个GUI的界面操作是更方便的。在开发管理后台项目的同时自己也能掌握`Vue+ElementUI`的技术栈组合使用

## 项目介绍

本项目使用`Vue+ElementUI`为核心技术开发，全部功能由个人单独开发完成。
本项目主要功能是管理客户端项目<http://123.56.124.33:5000>数据，包括各页面数据的增删改查，以及应用首页数据可视化的展现。

您也可以以此项目为模板进行二次开发，本项目的`UI`和`功能实现`均可作为参考。如果有兴趣进一步交流的同学欢迎到本项目仓库提`issues`，也欢迎您的收藏和关注，谢谢！

仓库地址：<https://github.com/konglingwen94/vue-seller-admin>

线上地址: <http://123.56.124.33:5000/admin>

## 部分应用页面截图
店铺配置页面
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db65994da7d74bbba666c17de91c4512~tplv-k3u1fbpfcp-watermark.image)

商品管理页面
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f0f473482244999bea218d897e8d14d~tplv-k3u1fbpfcp-watermark.image)

商品编辑页面

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1ac791f4cba4b8cb1d32dfcd42a8f75~tplv-k3u1fbpfcp-watermark.image)



## 应用开发技术

核心技术架构：`Vue@2.6.X+Element-ui@2.12.X`

依赖包：`vue-router` `v-charts`(基于echarts封装的vue组件库)  `axios(http请求工具)`

构建工具：`vue-cli`  `less` 

部署和运行：`github-actions` `云服务器` 


## 开发过程中的案例介绍

由于本应用使用场景较为广泛，这里有关业务场景的实现过程就不介绍了。在项目开发完成后我总结了几个常用场景下的技术实现想在这分享一下。如果你遇到了类型的功能需求可以参考下面的功能实现

### 二次封装uplod上传组件

组件源代码地址：<https://github.com/konglingwen94/vue-seller-admin/blob/master/src/components/v-upload.vue>

在多数的应用使用中我们都会遇到这样的需求，从本地电脑上传一张或多张图片到网站上，以上传的内容作为网站上需要为我们展现的内容。在开发这个功能的时候，对于用习惯`element-ui`里的`upload`组件的我来说的话，对于上传一张和多张图片不同的业务处理我们还可以定制封装`upload`组件，这样的细粒度处理以便于更符合我们的业务需求。

通常用的最多的就是嵌入到表单里的图片上传功能，除了输入框等类型的表单可以用`v-model`的形式处理数据外，`upload`同样也可以，这样就大大便利了在提交表单时对于数据的处理过程。

首先给`upload`组件定义这样几个`props`。

```
props: {
    value: [String, Array],
    multipart: { type: Boolean, default: false }
  },

```

看过`vue`官方文档自定义`v-model`指令的基本介绍的人都知道，`value`属性用于接收组件上`v-model`指令的绑定值，相应的在组件内通过`emit`一个`input`事件出去，传入的参数会自动更新到指令所绑定的值上。另外一个`multipart`属性则表明了组件可一次上传的文件个数。有了这两个`props`我们就有了大概实现整个组件功能的思路了。下面就可以基于这两个属性实现`emit`一个`input`事件的功能了。

对于所有的原生`el-upload`上文件更新的事件监听器，根据`multipart`属性值的不同，这里以`on-success`事件为例需要这样处理即可。

```js
// 这里仅为部分代码
handlerSuccess(res,file,fileList){
	if (!this.multipart) {
          if (fileList.length > 1) {
            fileList.shift();
          }
        }
        const value = this.multipart
          ? fileList.map(item => {
              return (item.response && item.response.path) || item.url;
            })
          : res.path;

        this.$emit("input", value);
}
```
至此动态更新组件`v-model`指令绑定值的逻辑就处理完了，然而经过我的测试发现还有一个组件初始化的过程需要处理，也就是对于绑定到组件上的`file-list`字段的初始化过程。首先需要在组件中定义一个`fileList`变量

```js
data(){
	return {
    	fileList:[]
    }
}
```
有了这个`fileList`变量我们就可以对基础的`upload`组件初始化数据了，具体的绑定过程这里就不演示了，详细操作可以查看官方的组件使用方式。但是经过测试这里还有一个问题，我们在表单里调用这个组件用`v-model`所绑定的值大多是以`API`请求的方式异步获取的，当请求完成时通过`v-model`指令传入的值并不能实时同步到`fileList`这个变量上，那要怎么解决这个问题呢？经过一番思考我想到了vue提供的`$watch`这个api，通过观测组件的`value`属性我们就可以解决这个问题了

```js
// 这里只需要在组件创建的时候一次性的观测`value`属性值的变化
created() {
    const unwatch = this.$watch("value", newVal => {
      this.fileList = Array(this.newVal)
        .flat()
        .map(item => ({ name: "", url: newVal }));

      unwatch && unwatch();
    });
  }

```

:::^ 备注
组件源代码:  <https://github.com/konglingwen94/vue-seller-admin/blob/master/src/components/v-upload.vue>

:::

现在再去测试组件发现可以完美的运行了

### 页面选项卡视图切换
效果图
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e90246e4ee004f70910a62a5ea98a95a~tplv-k3u1fbpfcp-watermark.image)

源代码 https://github.com/konglingwen94/vue-seller-admin/blob/master/src/layout/basic.vue

选项卡视图是类似于浏览器页面浏览的一种通过标签点击切换页面的操作方式，我们可以在这里查看点击过的页面，通过点击关闭按钮可以关闭相应的标签。选项卡组件自身的功能很简单，这里就不详细介绍了。下面主要聊聊点击页面后的交互效果怎么去实现。

既然有了选项卡，我们需要的就是把浏览过的页面有状态的保留下来，下次页面切换回来的时候还保留离开时的状态。说道这里我们很容易能想到vue自身提供的`keep-alive`组件，通过用这个组件包裹我们动态切换的路由组件就可以缓存项目渲染过的组件了。`keep-alive`具体的使用方式也不多说了，详情可以看官方文档介绍。

为了存储选项卡和缓存组件的数据我们需要定义两个数组类型的变量，通过对这两个数组的操作处理来达到具体功能的实现。

```html
/*
tab-tag  组件使用说明
title:HTML原生属性  String
click:监听标签点击事件  Function
close:监听标签关闭事件  Function
closable:是否可以关闭 Boolean
active: 当前是否激活  Boolean
label:展示文字，这里以插槽的方式传入  String
*/
// 这里只展示相关的代码片段

 <div class="tag-container">
          <tab-tag
            :title="tag.label"
            @click="toggleTag(tag)"
            @close="closeTag(tag, index)"
            v-for="(tag, index) in tags"
            :key="index"
            :closable="!tag.isHome"
            :active="$route.path === tag.path"
            :style="{ width: `${100 / maxTags}%` }"
          >{{ tag.label }}</tab-tag>
        </div>

        <el-main>
          <keep-alive :max="maxTags" :include="cacheViews">
            <router-view :key="$route.fullPath"></router-view>
          </keep-alive>
        </el-main>

```


```js
 data() {
    return {
      maxTags: 7,  // 选项卡最大存储个数
      tags: [
        {
          label: "首页",   // 标签内容
          name: "seller-dashboard",   // 对应页面路由的名称
          path: "/seller/dashboard",  // 对应页面路由的路径
          isHome: true
        }
      ],      //  选项卡初始化数据
      cacheViews: []   // 缓存的页面组件，存储组件内手动声明的`name`值
    };

```
有了上面在`data`函数里定义的变量，我们只需要把页面交互的功能实现就行了。每一次的页面切换应该怎么存储呢? 处理这个问题需要我们实时观测路由实例对象的变化，然后通过操作`tags`和`cacheViews`这两个变量以实现我们的需求

```js
watch: {
    $route(newRoute) {
      const index = this.tags.findIndex(item => item.path === newRoute.path);
      if (index === -1) {
        if (this.tags.length >= this.maxTags) {
          this.tags.splice(1, 1);
        }
        if (!this.cacheViews.includes(this.$route.name)) {
          this.cacheViews.push(this.$route.name);
        }
        this.tags.push({
          label:
            newRoute.meta.breadcrumbMenus[
              newRoute.meta.breadcrumbMenus.length - 1
            ],
          path: newRoute.path,
          name: newRoute.name
        });  // `meta.breadcrumbMenus`是我自己为了展示对应页面的面包屑菜单所设置的数据结构，你也可以定制自己的`label`字段。
      }
    }
  },
```

在存储路由缓存和选项卡标签的时候需要注意，我这里默认把设置的页面名称和路由名称保持一致，你也可以根据自己的需要灵活的控制，但是要保证`cacheViews`数组存储的一定是页面组件内定义的`name`值，这也是`keep-alive`组件在做缓存时所要求的，不然的话动态组件不会被其缓存

实现了缓存页面组件以及存储选项卡标签的功能后，对于选项卡标签的`切换`和`关闭`功能也就很简单了,通过分别监听选项卡标签得`click`和`close`事件来处理对应的逻辑

```js
methods: {
    closeTag(tag, index) {
      this.tags.splice(index, 1);
      const isActive = this.$route.path === tag.path;
      this.cacheViews.splice(this.cacheViews.indexOf(tag.name), 1);

      if (isActive && !tag.isHome) {
        this.$router.push(this.tags[this.tags.length - 1].path);
      }
    },
    toggleTag(tag, index) {
      if (tag.path === this.$route.path) {
        return;
      }
      this.$router.push(tag.path);
    }
  }

```

至此整个选项卡视图的功能就全部完成了，以上的介绍是我自己项目的实现方式，部分代码没有通用化处理，，本功能还有待优化的地方，后续完成后我会及时更新，当你碰到类似需求的场景开发时可以参照此实现方式，欢迎在评论区指点！

>参考：https://juejin.cn/post/6844903968125124616

## 项目中使用到的技术小魔法分享

### Vue的computed属性

我们都知道vue的`computed`属性在项目开发中无处不在，但是用的最多的是此属性的`getter`功能，深入了解`computed`属性的同学肯定知道他还有`setter`的功能可供我们使用，但是往往很多人不知道它的应用场景在哪，包括我自己在开发本项目之前也是对这个`setter`的使用场景了解很少。这里先介绍一下`computed`属性全部语法的书写方式

```js
// 这里以`formType`这个属性为例
computed:{
	formType: {
      set(val) {
        this.form.type = val === "" ? -1 : val;
      },
      get() {
        return this.form.type === -1 ? "" : this.form.type;
      }
}
```

代码中`formType`属性是一个拥有了`getter`和`setter`的计算属性。它的作用比较像js里面的`Object.defineProperty`方法，通过拦截对一个对象属性的读取完成一些特定的逻辑。可以看出对于`formType`属性的读取操作引发了另外一个数据的读取，这里就引出了`form.type`这个属性值了。了解了它的基本使用方式后就需要我们进一步来应用他了

```js
data(){
	return {
          form: { name: "", type: -1 }, // 因为实际项目表单项有多个，这里的`type`作为`form`对象的属性值定义

    }
}
```
从代码逻辑中不难看出，两段代码组合起来完成了`formType`和`form.type`关于`''`和`-1`这两个值响应式的特殊对应关系。那么这种特殊的一对一的数据转化对应关系究竟适合用到什么地方呢？这里我用了一个动态截图展示一下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c93af3e8ead48e99d124730dbf61dfa~tplv-k3u1fbpfcp-watermark.image)

先说明一下，表单中鼠标操作的选择框我用的以element-ui`select`类型的选择框，选择框表单项的所绑定的值就是`formType`

组件html
```html
<el-select v-model="formType" clearable placeholder="请选择优惠类型（可选）">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>

```

js部分
```js
data(){
	return {
          options: [
          { value: 0, label: "满减" },
          { value: 1, label: "折扣" },
          { value: 2, label: "特价" },
          { value: 3, label: "支持发票" },
          { value: 4, label: "外卖保" }
        ],
    }
}

```
从`js`部分的代码可以看出，表单项选择框里渲染的全部数据是这个样子的，选择框经过双向绑定后所取到的值是`value`这个字段。因为服务端的一些设计规则返给前端的大多是`number`类型的数据，而经过我们用计算属性处理后的`form.type`字段值也刚好符合要求。因此当我编辑完表单后，其数据类型完全符合后端的需要，这让前端提交请求接口时免去了需要重新处理数据结构不一致的问题，也加强了组件数据自身的响应化过程。

## 自动部署

本项目是使用`github-actions`提供的集成部署环境，本地代码提交更新后自动部署到云服务器。详细部署配置文件点这里 <https://github.com/konglingwen94/vue-seller-admin/blob/master/.github/workflows/main.yml>

## 管理员权限等级管理

对于比较大的管理后台系统会同时分配多个管理员，这些管理员之间也会有权限等级的分类。本项目为了练手此业务功能划分为了`超级`和`初级`两个等级，线上环境登录的默认是初级管理员，在管理员列表里可以看到。

不同等级的管理员只能查看同级或比自身等级更低的注册用户，超级管理员可以查看所有用户。
想要以超级管理员的方式登录需要配套后端<https://github.com/konglingwen94/elm-seller-server>项目使用`node`环境初始化自动[脚本](https://github.com/konglingwen94/elm-seller-server/blob/master/scripts/init-admin.js)后才可使用。


## 收获

通过本项目的开发自己在B端应用上也有了更多的经验和技术上的收获，通过对`Vue`以及其组件库在实际项目中不断的实践应用，自己对前端开发流程和规范也有了更多的思考。以前做好一个项目考虑的是怎么实现需求，怎么用更少的代码实现功能。而现在再让我去开发一个项目，我会更多的考虑怎么用另外一个思路完成同样的需求，同一个功能需求可能会有很多个答案，尝试更多的解决办法也能让自己在其他遇到的一些场景中用到，用一句话概括的话就是`不同的解决办法有异曲同工之妙`，这也会让自己的思路更开阔。

不光是在技术层面对`前端`有了更多的理解，在`开发规范`上自己也了解的更多了。比如代码的`可读性`，`维护性`，变量命名的`语义化`等等。就拿`代码可读性`来说,这个问题对于多人协作开发的项目来说是很重要的，怎么让别人能最快的靠`代码语义`读懂你的项目，现在我在应用实践中也会更多的考虑这个问题。在解决一个技术问题时，宁可让代码量变得多的，也要保证代码的`语义`和`可读性`更好，不能追求逻辑代码的精简（学习API练习demo的时候可以这么干）从而损害到了整个项目的`维护性`以及`代码可读性`问题。

## 项目预览

线上地址：<http://123.56.124.33:5000/admin>

github: https://github.com/konglingwen94/vue-seller-admin


### 其他关联项目

客户端线上地址：<http://123.56.124.33:5000>

客户端仓库地址：<https://github.com/konglingwen94/vue-elm-seller>

服务端仓库地址：<https://github.com/konglingwen94/elm-seller-server>


## 感谢

如果你对本项目有好的建议欢迎到仓库提[issues](https://github.com/konglingwen94/vue-seller-admin/issues/new)，欢迎小伙伴们交流留言，谢谢！

感谢所有阅读，点赞和关注的小伙伴们！
