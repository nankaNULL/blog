1. 没什么心情写，可是实在是没事情可以做了。

2. 目录(字体变大了，是我的错觉吗)

   (1) 项目架构

   (2) Router

   (3) controller

   (4) service

   (5) config 配置

3. 待解决：

   (1) controller用作接口的那个，有点问题，长卿项目上用的api那个地方有点迷

4. 项目地址：

   <https://github.com/nankaNULL/nodejs-egg>

------

项目架构

```
├── config // 配置文件
│     └── config.default.js
│
├── app //项目的主要目录
│     ├── controller //控制器
│     ├── public //静态文件
│     │     ├── js 
│     │     └── css 
│     ├── view //页面
│     ├── extend //扩展插件
│     ├── service //http请求
│     └── router.js //路由
│
└── package.json //node相关环境的配置文件
```

------

Router

1. router.js这个文件下存放的都是路由地址，相当于是一个路由表，对应每一个接口是哪个

   ```
    router.get('/news/:id', controller.news.list);
   ```

2. 一句话囊括三个要点

   (1) get/post等 请求方式

   (2) '/news/:id' 路由地址

   (3) controller.news.list 对应到指定的控制器上

------

Controller

1. 文档上是这么讲的

   “简单的说 Controller 负责解析用户的输入，处理后返回相应的结果”，

   “框架推荐 Controller 层主要对用户的请求参数进行处理（校验、转换），然后调用对应的 service方法处理业务，得到业务结果后封装并返回”

2. 然后文档还列出了适用场景（<https://eggjs.org/zh-cn/basics/controller.html>）

3. 我不想抄了，链接给出。总之controller层就是对数据进行处理，这个数据可以是通过service，调用http请求拿到，而后对数据处理，返回。

4. 这个返回数据又可以是ctx.render渲染，（例如文档给出的例子）

   也可以只是返回数据，当接口用（例如长卿的doraemon）

5. 写法上，是通过定义类的方式，继承Controller类来编写

6. 有几个属性会挂在this上（这几个属性，service也有的）

   (1) this.ctx：当前请求的上下文context对象的实例

   (2) this.app：当前应用application对象的实例，通过它我们可以拿到框架提供的全局对象和方法。

   (3) this.service：应用定义的service, 通过它我们可以访问到抽象出的业务层

   (4) this.config：配置

   (5) this.logger：logger对象，用以打印日志（方法有debug, info, warn, error）并对应打印文件路径，（可以替掉console了）

7. 然后这里有一点我不是很明白，文档说controller 基本上是业务开发中唯一和 HTTP 协议打交道的地方，但我之前的理解都是http的请求响应是放在service的啊，而且service也是同样可以拿到this.ctx.query的呀

------

Service

1. 复杂数据的处理
2. Controller 一般不会自己产出数据，也不会包含复杂的逻辑，复杂的过程应抽象为业务逻辑层 Service

------

config 配置

1. 这个配置是个对象，感觉还是拆开来写的好
2. 框架支持根据环境来加载配置，config.default.js这个是默认配置，prod环境下回加载config.prod.js和默认的，并覆盖默认中的同名配置
3. 另外还有个插件的配置，放在plugin.js下（但有时候config.default.js里也要对应配置？emmm在看吧）