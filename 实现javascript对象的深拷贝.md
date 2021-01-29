
文章首发于[个人博客](https://github.com/konglingwen94/myproject-articles-summary)

## 前提

在处理日常的业务开发当中，数据拷贝是经常需要用到的。但是 javascript 提供的数据操作 Api 当中能实现对象克隆的都是浅拷贝，比如 Object.assign 和 ES6 新增的对象扩展运算符（...）,这两个 Api 只能实现对象属性的一层拷贝，对于复制的属性其值如果是引用类型的情况下，拷贝过后的新对象还是会保留对它们的引用。

## 简单粗暴的深拷贝

ESMAScript 给我们提供了关于操作 JSON 数据的两个 APi，通过把 javascript 对象先转换为 JSON 数据，之后再把 JSON 数据转换为 Javascript 对象，这样就简单粗暴的实现了一个 javascript 对象的深拷贝，不论这个对象层级有多深，都会完全与源对象没有任何联系，切断其属性的引用，现在看一下这两个 API 用代码是怎么实现的。

```javascript
// 现创建一个具有深度嵌套的源对象
const sourceObj = {
  nested: {
    name: 'KongLingWen',
  },
}

// 把源对象转化为JSON格式的字符串

const jsonTarget = JSON.stringify(sourceObj)

// 以解析json数据的方式转换为javascript对象
let target
try {
  target = JSON.parse(jsonTarget) //数据如果不符合JSON对象格式会抛错，这里加一个错误处理
} catch (err) {
  console.error(err)
}

target.nested.name = ''

console.log(soruceObj.nested.name) //KongLingWen
```

代码最后通过更改新拷贝对象的 name 属性 ，输出源对象此属性的值不变，这说明我们以这种方式就实现了一个对象深拷贝。

## JSON.parse 和 JSON.stringify 转换属性值前后的不一致性

* 函数无法序列化函数，属性值为函数的属性转换之后丢失
* 日期 Date 对象
  javascript Date 对象转换到 JSON 对象之后无法反解析为 原对象类型，解析后的值仍然是 JSON 格式的字符串
* 正则 RegExp 对象
  RegExp 对象序列化后为一个普通的 javascript 对象，同样不符合预期
* undefined
  序列化之后直接被过滤掉，丢失拷贝的属性
* NaN
  序列化之后为 null，同样不符合预期结果

此方式拷贝对象因为有以上这么多缺陷，所以我们不如自己封装一个属于自己的 javascript 对象深拷贝的函数，反而一劳永逸。

## 手动封装对象深拷贝方法

对象属性的拷贝无疑就是把源对象的属性以深度遍历的方式复制到新的对象上，当遍历到一个属性值为对象类型的值时，就需要针对这个值进行再次的遍历，也是就用递归的方式遍历源对象的所有属性。让我们先看这一部分代码

```javascript
function cloneDeep(obj) {
  const result = {}
  for (let key in obj) {
    // 判断key 是否是对象自身上的属性，以避免对象原型链上属性的拷贝
    if (obj.hasOwnProperty(key)) {
      result[key] = cloneDeep(obj[key]) //需要对属性值递归拷贝
    }
  }
}
```

这段代码是对象属性深拷贝的逻辑，但是不同的数据类型各自有其特殊的操作方式需要处理，下面就把这些处理边界场景的代码补充上，看看完成的代码应该是怎样的

```javascript
function isPrimitiveValue(value) {
  if (
    typeof value === 'string' ||
    typeof value === 'number' ||
    value == null ||
    typeof value === 'boolean' ||
    Number.isNaN(value)
  ) {
    return true
  }

  return false
}

function cloneDeep(value) {
  // 判断拷贝的数据类型，如果为原始类型数据，直接返回其值

  if (isPrimitiveValue(value)) {
    return value
  }
  // 定义一个保存引用类型的变量,根据 引用数据类型不同的子类型初始化不同的值，以下对象类型的判断和初始化可以根据自身功能的需要做删减。这里列出了所有的引用类型的场景。
  let result

  if (typeof value === 'function') {
    // result=value 如果复制函数的时候需要保持同一个引用可以省去新函数的创建，这里用eval创建了一个原函数的副本
    result = eval(`(${value.toString()})`)
  } else if (Array.isArray(value)) {
    result = []
  } else if (value instanceof RegExp) {
    result = new RegExp(value)
  } else if (value instanceof Date) {
    result = new Date(value)
  } else if (value instanceof Number) {
    result = new Number(value)
  } else if (value instanceof String) {
    result = new String(value)
  } else if (value instanceof Boolean) {
    result = new Boolean(value)
  } else if (typeof value === 'object') {
    result = new Object()
  }

  for (let key in value) {
    if (value.hasOwnProperty(key)) {
      try {
        result[key] = cloneObject(value[key]) //属性值为原始类型包装对象的时候，（Number,String,Boolean）这里会抛错，需要加一个错误处理，对运行结果没有影响。
      } catch (error) {
        // console.error(error)
      }
    }
  }

  return result
}
```

代码中首先封装了一个判断数据是否是原始类型的方法，这里只是为了保持 cloneDeep 函数的功能干净，其实你也可以完全放到一块，这个完全取决于自己的编码风格。如果在业务上需要有多处判断数据是原始类型还是引用类型的场景时，以上这种代码功能抽离的方式就方便处理了。

查看更多 Javascript 函数功能封装在 Github 仓库[狠狠的点击这里](https://github.com/konglingwen94/handy-utilities)

备注：本片博文为作者原创，转载请标注出处，谢谢！

作者：孔令文
