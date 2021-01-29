 
## 基本介绍

我们都知道浏览器是没有原生的 DOMResize 事件的，唯一能监测到 Resize 事件的对象就是 window 这个全局对象，但是这个事件只有在窗口大小改变的时候才会触发。当我们需要在 DOM 元素尺寸变化时想做一些事情的话就显得无能为力了，那么我们今天就来聊一聊如何手动实现这个功能，其实也就是在给 DOM 元素增加一个 Resize 事件侦听器的 polyfill。

## DOM 事件是怎么触发的

在实现这个功能之前我们先回顾一下平时给浏览器 DOM 注册各种各样的事件是怎么触发的呢?如果我们把这个问题搞明白了，这对接下来要实现的功能就能打开一个很好的思路。

原生的 DOM 事件是浏览器响应用户的交互行为，然后内部会自动派发这个事件，当我们给 DOM 注册的事件被浏览器派发是，先前绑定的回调函数就会执行了。首先注册一个 DOM 事件是很容易的，就是我们经常写的代码

```javascript
document.querySelector('selector').addEventListener('resize', handler)
```

理想的情况是当我们改变元素尺寸是，注册侦听器函数的第二个参数 handler 就会被执行，当然这也是我们这篇文章最终要实现的目标。

## 如何手动创建一个 DOM 事件

如果我们能自己手动创建一个 DOM 事件，然后在需要被触发的时候去派发它，这样我们的目的就达到了。当然这就要求我们对浏览器事件的相关 Api 有一定的了解，让我们直接来看下面的代码吧。

```javascript
const event = document.createEvent('HTMLEvents')

event.initEvent('resize')
```

这样简单的两行代码就创建了一个自定义 DOM 事件，至于代码中两个函数的可传参数都有哪些可以去翻阅文档具体了解，本文就不在赘述了。

## 观测 DOM 元素尺寸变化（功能实现的核心）

我们有了自定义的事件，至于要我们在适当的时候派发它就可以实现注册函数的自动回调。那么这个事件派发的时机我们又要怎么完成。这就引出了本文的重点：如何观测 DOM 元素的尺寸变化？幸好我们使用的现代浏览器提供了一个观察元素尺寸变化的 Api(ResizeObserver),有个这个接口，我们就可以进行代码实现了

```javascript
const observer = new ResizeObserver(function elResizeChange(entries) {
  // 每次被观测的元素尺寸发生改变这里都会执行
})
observer.observe(el) // 观测DOM元素
```

从代码里我们可以看到，我们先是以构造器的方式创建了一个 observer 实例，然后又调用了实例的一个方法去观测元素，这样我们就实现了一个 DOM 元素的尺寸观测方法。接着我们要说到传递到构造器里面的那个回调函数的作用了，其主要功能都是在这个函数里实现的。这个函数会在被观测元素尺寸变化的时候被调用，接下来我们也会在这里去实现事件的派发功能

## 事件的派发

接着上面已经实现的代码我们继续来实现 DOM 自定义事件的派发功能，这也是实现 Resize 事件的最后一步，等实现了这一部，Resize 事件的实现就大功告成了！同样让我们直接来看代码吧

```javascript
// 这里的代码截取自上面的回调函数
function elResizeChange(entries) {
  // entries 是个数组 每个成员又是一个ResizeObserverEntry 实例

  for (let entry of entries) {
    const event = document.createEvent('HTMLEvents') //创建DOM事件
    event.initEvent('resize') // 初始化事件类型为resize
    entry.target.dispatchEvent(event) // 事件派发
  }
}
```

这里的 elResizeChange 函数就是我们创建元素尺寸观察者实例时传入的回调函数,在这里我们实现了元素自定义事件的派发。我们在重点讲一下这个函数主要实现的功能。

因为一个 observer 观察者实例（一个 window 窗口仅需一个 observer 实例）需要观测多个元素的尺寸变化，所以回调函数的入参是一个数组类型的数据结构，每一个元素成 员都是一个 ResizeObserverEntry 实例（包含被观测元素尺寸变化信息的一个对象）。代码中的最后一行使用此实例的 target（指向尺寸变化的目标元素）属性进行事件派发功能。至此整个 DOM Resize 事件的自定义侦测功能我们就做好了，下面就让我们把它合并到浏览器原生的事件接口中去吧!

## 封装事件到浏览器内置接口

手动实现 DOM Resize 事件的功能我们已经做出来了，但是我们在给 DOM 元素注册事件的时候怎么才能知道需要去观察它的尺寸呢？这就需要我们对注册事件的程序进行拦截，在注册事件之前加入自己的处理逻辑，然后再包装成一个全新的事件侦听器，下面让我们直接来看代码吧

```javascript
const addEventListener = EventTarget.prototype.addEventListener

HTMLElement.prototype.addEventListener = function _wrapper(type) {
  if (type === 'resize') {
    registerEvent() //讲到这里，聪明的你一定能实现这个函数，赶快动手去实现吧！
  }

  addEventListener.apply(this, arguments)
}
```
首先我们对原有的DOM事件注册侦听器做一个缓存，然后我们手动包装一个事件侦听器函数覆盖到HTMLElement的原型对象上，这里简单说明一下为什么是覆盖到HTMLElement而不是Element,了解原型链的同学应该都知道，子类会继承从父类到原型链末梢的所有属性和方法，这里如果把事件侦听器重写到Element的原型上的话，文档中的所有节点类型都会继承这个方法，就连文本节点上的原型方法也被覆盖了，这就造成了不必要的功能浪费，所以这里只需让HTML元素的节点注册侦听器生效即可。

重写的事件侦听器包装函数内部通过判断当前注册的事件类型实现自定义的事件注册功能即可，这里以resize 事件拦截为例子，然后把前面讲到的自定义事件功能实现的逻辑放到这里即可，代码中以registerEvent这个函数为案例，你还可以实现其他的自定义事件。

文章的最后附上本项目的Github仓库地址[点击查看](https://github.com/konglingwen94/element-resize-event-polyfill)

备注：本篇文章为作者原创，转载请注明出处，谢谢！