
## this 是什么

Javascript`this`关键词指的是他所属的对象，它拥有不同的值，具体取决于使用的位置和调用方式。

- 使用方式
  - 在方法中，它指向这个方法的`拥有者`
  - 在函数中，它是全局对象`window`
  - 严格模式下在函数中，它是`undefined`
  - 单独使用时，它是全局对象`window`
  - 在事件中，它指向触发事件的目标对象`e.target`

## 不同执行模式下的差异化

javascript 中的`this`不同于其他编程语言，在严格模式和非严格模式下， 它的值是不同的，下面举个例子。

```javascript
// 非严格模式

function foo() {
  return this
}

console.log(foo() === window.foo()) //true

// 严格模式
;('use strict')

function foo() {
  return this
}

console.log(foo()) // undefined
console.log(window.foo()) // window
```

非严格模式下`foo`不论是函数调用还是作为一个方法调用，内部`this`的指向都是`window`。
在严格模式下，`foo`作为一个函数调用时`this`的值为`undefined`,而当做`window`对象的一个方法调用时，它的值指向了调用它的`window`对象。从代码书写的语义化来看的话，这种`this`的指向会更合理，这也是`javascript`的执行环境逐渐向严格模式靠拢的原因，从语言层面抛除一些不符合预期的执行结果。

## 灵活使用`this`手动实现函数的`call`和`apply`

了解了`this`的诸多特性后我们能利用它实现什么有趣的功能呢？没错，就是下面要手动实现函数的`call`和`apply`方法。

首先我们需要知道`es5`的`call`方法实现了什么功能，它的第一个参数是调用函数的`this`绑定，其余的参数会作为实参传递给执行的函数。了解了这个功能，我们就可以动手去实现它了。

```javascript
Function.prototype._call = function _call(context, ...args) {
  if (context == undefined) {
    context = window || global
  } else {
    context = Object(context)
  }
  // 获取当前要调用的函数挂载到`context`上

  context.handler = this
  const result = context.handler(...args)
  delete context.handler
  return result
}

// 测试一下
const value = 10

const foo = {
  value: 1,
}

function print(arg) {
  console.log(this.value)

  return arg
}

const retVal = print._call(foo, 2) // 1
console.log(retVal) // 2
```

为了让`_call`函数的第一个参数作为`this`传入到执行函数中去，这里在`_call`函数内部做了一个变通，把待执行的函数作为`context`对象的一个方法调用，这样就实现了更改函数调用时`this`指向的问题。测试后的执行结果也是符合我们的预期的。同理`apply`函数的实现方式也是如此，只需要把传递的参数处理一下即可，这里就不在演示了。

备注: 本篇文章属于作者原创，转载请注明出处。<br>
参考：[JavaScript 深入之 call 和 apply 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)
