
## 背景

自从完成了[客户端](https://github.com/konglingwen94/vue-elm-seller)和[管理后台](https://github.com/konglingwen94/vue-seller-admin)项目后，一个完整的`web`应用前端方面的项目算是搭建完成了。最后还需要有一个提供API服务的后端项目服务前端应用运行，经过一个月的开发，现在基本上已经全部完成，经过部署后，完成了最后的上线运行。

## 项目使用技术栈

本项目是基于`nodejs`主要使用`koa+mongodb`为核心开发的轻量级服务端应用。接口是按照`RESTful`风格进行设计


使用主要中间件有
`koa-compress`,		
`koa-parameter`,`koa-connect-history-api-fallback`,`koa-static`,`koa-mount`。具体使用方式请在`koa`官方仓库[查看](https://github.com/koajs)

数据库操作：`mongoose`

接口权限验证：`jsonwebtoken`

用户密码加密：`bcryptjs`

上传资源存储：`koa-multer`

路由分发：`koa-router`

接口参数解析: `koa-bodyparser`


>接口使用文档:	<https://konglingwen94.github.io/elm-seller-server>


## 开发过程

### 数据库设计
本项目选择使用`mongodb`作为数据存储的数据库，因为其对于前端开发者有着天然的有好性，使用容易上手。
由于本人涉猎服务端领域尚处于初级阶段，在数据库设计方面经验有限，在此分享出来仅供参考使用。

`mongodb`使用`bson`类型作为数据存储格式。由于跟前端js的`json`类型可以互相转换使用。所以这就减小了入门者设计数据库表字段的难度，我们可以参照前端页面需要展示的数据进行设计。
然后通过使用`mongoose`这个库作为快速操作数据库的模型，我们可以用代码的形式设计`mongodb`表字段的模型，经过`mongoose`的编译可以存储到真实的数据库表中。

拿本项目中的`商品`表来说，这是一个声明好的mongoose`Schema`


```json
 {
    name: String,
    price: Number,
    oldPrice: Number,
    description: String,
    sellCount: Number,
    rating: Number,
    info: String,
    menuID: ObjectId,
    image: String,
    online: { type: Boolean, default: true },
  },

```
通过`mongoose`模型的编译方法后存储到数据库中的字段是这个样子

#### 数据库存储字段

|    Field    |   Type   | Description |
| :---------: | :------: | :---------: |
|   menuID    | ObjectId | 商品分类 ID |
|    name     |  String  |  商品标题   |
|    info     |  String  |  商品信息   |
| description |  String  |  商品简介   |
|    image    |  String  |  商品封面   |
|   online    | Boolean  |  是否发布   |
|  oldPrice   |  Number  |  商品原价   |
|    price    |  Number  |  商品售价   |
|  sellCount  |  Number  |  售卖个数   |

>查看完整的模型文件点[这里](https://github.com/konglingwen94/elm-seller-server/blob/master/model/food.js)

### 接口搭建

一个完整的API接口从接收请求到响应数据完成，中间这个过程就是服务端处理各种代码逻辑的。这其中主要包括`暴露接口地址`,`接口权限验证`，`请求参数验证`，`查询数据库`，`返回响应信息`这几个阶段。为了符合服务端业务逻辑分层设计的模式，每一个处理阶段都可以抽离到一个单独的模块，最后再把各种相关联的模块组装起来打包成一个完整的项目，这样的模块化设计可以很大的增强项目的`维护性`和`可读性`。用目录结构的方式展现就是这个样子的

```bash
├── model  // 数据库模型
│   ├── administrator.js
│   ├── seller.js
│   ├── rating.js
│   ├── category.js
│   └── food.js
├── helper
│   ├── validatorRules.json  // 参数验证规则
│   ├── mongoose.js  // mongoose连接脚本
│   ├── middleware.js // 项目中间件
│   └── util.js  // 工具函数
├── controller  // 控制器
│   ├── administrator.js
│   ├── seller.js
│   ├── rating.js
│   ├── category.js
│   └── food.js
├── config
│   └── config.default.json  // 项目配置文件
├── router
│   └── index.js  // 路由配置

```
`model`文件夹用来放数据库表模型，数据库存储了哪些字段在这个文件目录查看一目了然。`helper`目录存放了一些辅助的项目文件和一些脚本，其中`middleware.js`这个文件存放了整个项目的所有中间件，按照模式分层的原则，我把服务端接口的一些处理逻辑都抽离到了中间件里，其中包括`接口权限验证`，`请求参数验证`这两个主要的代码处理逻辑。`controller`目录则是存放接口业务逻辑的地方，我们也把他叫做`控制器`，`查询数据库`和`返回响应信息`也是在这个模块里面完成的。最后就是统一分发路由接口，`router`目录是项目所有接口分发的地方，在这里可以把不同的控制器分发到一个或多个路由接口地址上，这样可以实现控制器文件的复用，不需要写重复的业务代码。

## 权限验证和登录（包含注册功能）

面向多用户服务的后端项目，权限验证是不可或缺的。本项目使用了`authorization`请求头验证的方式判断每一个请求的权限。为了方便处理，我把这一块的代码逻辑抽离到了一个中间件里。这样对每一个接口是否验证权限也容易管理和阅读。本项目的权限验证使用`jsonwebtoken`这个第三方插件作为生成秘钥`token`的工具，用户在登录的时候服务端会生成一个`token`响应到前端，前端根据运行环境把它存储下来，之后的每一个请求根据业务需要携带这个`token`传递到服务端，服务端根据设置好的验证规则返回不同的验证结果，这就是本项目`接口权限验证`整体运行过程。

通过[控制器中的用户登录接口]()分析其中的业务逻辑时怎么处理的

```js
// 这里仅展示业务逻辑代码
  async login(ctx) {
    const { username, password } = ctx.request.body;

    let result = await AdministratorModel.findOne({ username });
    //如果没有结果则 创建新用户
    if (!result) {
      // 加密密码
      const hashPass = await bcrypt.hash(password, 10);

      const newUser = await AdministratorModel.create({ password: hashPass, username });

      const token = jwt.sign({ username, role: newUser.role, level: newUser.level }, secretKey, {
        expiresIn,
      });

      return (ctx.body = { admin: omit(newUser.toObject(), ["password"]), token });
    }

    if (!bcrypt.compareSync(password, result.password)) {
      ctx.status = 400;

      return (ctx.body = { message: "密码错误" });
    }
    const user = result.toObject();
    const token = jwt.sign(user, secretKey, { expiresIn });
     
    ctx.body = { admin: omit(user, ["password"]), token };
  },
```
为了支持[管理后天](https://github.com/konglingwen94/vue-seller-admin)`首次登陆即注册`的功能，本登录代码接口也包含了用户注册的业务逻辑。经过参数解析和校验的过程后（代码部分以中间件处理的方式在其他模块），通过解构即取到了前端传递的有效参数。根据数据库查询的结果处理不同的业务逻辑，在取到创建后的用户信息后需通过`jsonwebtoken`的签名生成一个`token`，此`token`也是其他接口在验证用户登录状态时唯一的验证信息。


通过登录接口生成`token`后我们就可以对其他需要添加访问权限的接口进行鉴权验证了。下面是通过验证`token`是否有效判断用户登录状态的逻辑代码中间件

```js
// 统一抽离到一个中间件中，这里省略了引入其他模块的过程
module.exports={
 adminRequired() {
    return async (ctx, next) => {
      let token = ctx.headers["authorization"];

      if (!token) {
        ctx.status = 400;
        return (ctx.body = { message: "没有传递token" });
      }
      token = token.split(" ")[1];

      try {
        var decodeToken = jwt.verify(token, secretKey, { expiresIn });
      } catch (error) {
        ctx.status = 403;
        if (error.name === "TokenExpiredError") {
          return (ctx.body = { message: "过期的token" });
        }
        return (ctx.body = { message: "无效的token" });
      }

      ctx.state.adminInfo = decodeToken;
      await next();
    };
  },
 }
```
请求进入到这里后，通过认证请求头先提取到`token`变量，当`token`取到具体值后，再使用`jsonwebtoken`内部提供的验证函数校验，根据不同的验证结果响应不同的状态码和错误信息。具体的验证结果错误类型自行到插件仓库查看，这里不做详细介绍。当验证通过后会解析出`token`的签名内容，如果鉴权接口其他地方的业务逻辑需要用到此信息的话，我们可以把它挂载到`koa`提供的特定命名空间字段上，这样方便局部的逻辑代码获取。

>备注：`token`使用的认证类型需要根据前后端开发人员的约定使用，本项目使用`Bearer ${token}`的格式作为令牌访问头

为了符合`koa`中间件导出格式的设计原则，这个文件的中间件是以闭包的形式导出的，实际应用到接口上的是这个闭包函数，这样设计的好处是我们在调用中间件函数的时可以传递参数进去，内部实际生效的中间件可以根据外部传递的参数做逻辑上的处理。在路由配置表里面统一使用这个中间件的方式是这个样子的


```js
//部分代码省略
const Router = require("koa-router");

const router = new Router({ prefix: "/api" });
const middleware = require("../helper/middleware");

router.post("/admin/foods", middleware.adminRequired(),FoodController.createOne);

```

## 数据库分页查询功能

对于大多数前端项目，分页显示数据在一个非常常见的功能，对应到服务端的代码逻辑就是数据库的过滤查询。使用`mongoose`提供的过滤查询操作`API`可以很容易完成这个需求，当我们用到的地方比较多的时候，问题就出现了。对于前端请求的接口路径一般是这个样子的`/api/foods?page=1&size=20`，我们需要对传递的`querystirng`做进一步的判断和解析才能应用到数据库参数的查询上。问题是很多个接口都需要这个功能，使用起来比较繁琐，那不如我们把这个解析查询参数的过程抽离成一个模块，这样更方便我们使用和维护。现在让我们看一下封装好的全部代码吧！

```js
module.exports = {
  resolvePagination(pagination = {}) {
    const defaults = { page: 1, size: 10 };

    pagination.page = parseInt(pagination.page, 10);
    pagination.size = parseInt(pagination.size, 10);

    if (Number.isNaN(pagination.page) || pagination.page <= 0) {
      pagination.page = defaults.page;
    }
    if (Number.isNaN(pagination.size) || pagination.size <= 0) {
      pagination.size = defaults.size;
    }

    const { page, size } = pagination;
    return {
      page,
      size,
    };
  },
  resolveFilterOptions(filter = {}) {
    let sort = {
      createdAt: -1,
    };
    sort = defaults({}, filter.sort, sort);

    const { page, pageSize } = resolvePagination({
      page: filter.page,
      size: filter.size,
    });
    return {
      limit: size,
      skip: (page - 1) * size,
      sort,
    };
  },
};

```

首先通过`resolvePagination`这个函数我们可以解析出有效的`query`参数，在通过`resolveFilterOptions`这个函数解析出来符合`mongoose`数据筛选操作的查询选项。通过模块化引入的操作方式，应用到实际的数据库查询过程中如下

```js
// 代码片段来自项目`controller`目录
const {  resolveFilterOptions, resolvePagination } = require("../helper/utils");

module.exports={
 async queryListByOpts(ctx) {
    const { page, size } = resolvePagination({ page: ctx.query.page, size: ctx.query.size });

    const { skip, limit, sort } = resolveFilterOptions({ page, size });

    const total = await FoodModel.countDocuments();

    var results = await FoodModel.find().populate("category").sort(sort).skip(skip).limit(limit);

    ctx.body = {
      data: results,
      total,
      pagination: {
        page,
        size,
      },
    };
  },
}
```
从代码中可以看到以获取到前端传递的`query`类型参数为解析值，`resolvePagination`函数负责解析有效的数据分页查询选项，`resolveFilterOptions`函数解析出来了`mongoose`特定查询语句格式的参数，我们通过分离业务代码和逻辑代码的方式有效增强了代码的模块化结构，也增加了代码的复用性，提高了项目的开发效率。

## 应用的部署和运行

本项目使用`github-actions`的持续集成功能自动部署到云服务器，有了持续集成的服务，就省去了项目手动构建，测试，发布这一系列流程，而且降低了手动操作程序出错的风险，具体的配置文件如下

```yml
name: Deploy files
on: [push]
jobs:

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: copy file via ssh key
      uses: appleboy/scp-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USERNAME }}
        key: ${{ secrets.SERVER_SSH_KEY }}
        port: ${{ secrets.SERVER_PORT }}
        source: "*"
        target: "/var/www/elm-seller-server"

    - name: executing remote ssh commands using ssh key
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USERNAME }}
        key: ${{ secrets.SERVER_SSH_KEY }}
        port: "22"
        script: |
          cd /var/www/elm-seller-server
          npm install
          
          npm start
          
```

从配置文件中看出，服务器发布的环境变量都采用了加密的方式传递，比如`${{secrets.SERVER_HOST}}`这个环境变量，真实的存储值需要我们在`github`仓库的设置面板里的`secret`选项配置的，当本地使用`git`管理的仓库推送到远程仓库的时候就会触发`github-actions`的自动部署操作，同时我们还可以在`workflows`文件夹下面配置多个以`.yml`结尾的配置文件，一个配置文件对应一个`actions`部署任务，本项目我就使用了两个持续集成的任务，因为项目对应的[说明文档](https://konglingwen94.github.io/elm-seller-server)也需要及时的更新发布。至于部署文件模板怎么选择需要根据个人的需求自己选择设置，`github-actions`[官方市场](https://github.com/marketplace)提供了常用的集成任务模板供我们选择



发布到云服务器的应用我选择使用`PM2`管理应用，应用启动的配置文件[点这里](https://github.com/konglingwen94/elm-seller-server/blob/master/ecosystem.config.js)。pm2是一个面对`node`应用的管理工具，我们可以方便的查看，重启，删除，停止，启动应用

## API文档编写

文档的撰写是一个后端项目不可或缺的一部分内容，学会写文档可以回顾项目从设计到开发的过程，发现有问题的地方可以第一时间发现，及时的修复`bug`。项目文档是使用`markdown`语法编写的`REAEME`文件，所有文件均在项目的`docs`目录内。文档使用`vuepress`作为构建工具预览和发布。具体使用方式自行查看官方文档，不做详细介绍
>文档发布地址：<https://konglingwen94.github.io/elm-seller-server>

## 工具和环境

`vscode` `mac` `node` `mongodb` `git` `github` `postman` `ssh`


## 总结心得

从项目的需求规划，到数据库表设计，api接口逻辑关注点的分离，最后成功的部署运行以及文档的撰写完成，自己初步掌握了服务端项目完整的开发流程，并积累了一些开发经验可以在这分享。

作为编程开发人员，在项目开发过程中遇到困难是很正常的，尤其是在调试代码的时候各种各样的错误信息看的"眼花缭乱",尤其是服务端`node`的环境没有浏览器客户端调试方便。遇到代码出错不要怕，我们需要一步步排查出错的原因，如果错误信息看起来不直观我们可以借助第三方工具调试，本项目我使用的是`nodemon`这个工具，他可以热加载应用，也可以开启`debug`的命令打开一个类似浏览器开发者工具的调试面板，我们可以在控制台面板查看程序抛出的错误信息，在`source`面板查看出错的代码堆栈，借助这些工具的分析，只要有耐心，一点点思考出现错误的问题，最终我们一定可以解决它。

## 支持

感谢所有点赞和关注的小伙伴们，对本项目有兴趣的同学可以一块和我交流，欢迎在下面留言！

如果您对本项目由好的建议或者发现`bug`可以到项目仓库提`issues`,也欢迎您的收藏和关注，谢谢！

仓库地址:<https://github.com/konglingwen94/elm-seller-server>

文档地址：<https://konglingwen94.github.io/elm-seller-server>

