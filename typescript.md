typescript-react之悲壮的踩坑之路

目录：

1. 前期

2. typescript的相关配置

   (1) webpack

   (2) tsconfig

   (3) package.json

3. 开始踩坑

   (1) 替换后缀

   (2) 类型定义文档

   (3) 类型检查

   (4) antd

4. TypeScript - React

   (1) 接口

   (2) 类

问题待解决：

1. typescript 类型有哪些没写

项目地址：

​	<https://github.com/nankaNULL/typescript-react-webpack>

------

### 前期

1. 一开始真的很懵逼啊，打开了官网 -> 起步 -> 点击react -> 到了github???? -> create-react-app起步？？(这谁遭得住啊)，然后百度找教程，终于找到一篇不错的，嗯，开始踩坑

2. 参考文档以及项目

   https://www.cnblogs.com/jiasm/p/9542253.html

   https://www.tslang.cn/docs/handbook/react-&-webpack.html

   https://github.com/dtux-kangaroo/pc-react-ant.design

   https://github.com/Microsoft/TypeScript-React-Starter#typescript-react-starter

------

### typescript的相关配置

1. 先尝试替换一个影响不大的组件，前面balabala一堆的跳过，嗯终于看到一句醒脑的话，

   typescript的相关配置

2. ts的配置分为两块，一个是webpack里的配置，还有一个是tsconfig的配置

##### webpack

1. 首先是webpack里，对ts,tsx文件的解析，用到ts-loader 和 babel-loader，将ts中的一些特性转为最终可执行的js代码。好，开始装包

   ```
   yarn add ts-loader -D
   ```

2. 然后在对应开发环境中配上

   ```
   module:{
   	rules: [{
           test: /\.tsx?$/,
           use: ['babel-loader', 'ts-loader'],
           exclude: /node_modules/,
   	}] 
   }
   ```

3. 另外还有两个地方需要更改

   一个是resolve 下的 extensions，要注意添加上.ts和.tsx让他自行去匹配，不然会Can't resolve

   还有一个是entry入口文件，我之前是直接写成了main.js，如果是没加后缀的话，这里就不用改了

##### tsconfig

​	这个配置巨多，还巨看不懂，直接是拿过来超了反正，然后有几个坑注意一下

1. 要使用修饰器

   connect不是被改写成修饰器的写法了吗，@connect(state,dispatch)，要使用修饰器的话，tsconfig要配置两个属性:

   ```
   // 开启装饰器的使用
   "experimentalDecorators": true,
   "emitDecoratorMetadata": true,
   ```

2. webpack别名（alias）找不到路径问题，

   ```
   // import的相对起始路径 
   "baseUrl": ".",
   
   // paths下的路径，别名，和webpack下的对应
   "paths": {
       "@/*":["./src/*"],
       "public/*": ["./public/*"],
       "components/*": ["./src/components/*"],
       "pages/*": ["./src/pages/*"],
       "layout/*": ["./src/layout/*"]
   },
   ```

3. 还有几个比较重要的属性

   (1) target 指定ECMAScript的版本 

   ​	官方给的是ES5，但这会导致一些ES6的语法不兼容无法使用，

   ​	解决方式：1. 直接改成ES6 

   ​			  2. 保持ES5不变，添加lib:['es6','dom'] // 这里配置，需要async的话还可以改成es7	

   (2) module 指定module版本

   (3) strict 严格模式，这个到底要不要开启，，emm

   (4) sourceMap 编译文件对应关系

   (5) removeComments 编译生产的JavaScript文件是否包含注释

   (6) noImplicitAny 为 false 时，如果编译器无法根据变量的使用来判断类型时，将用 any 类型代替。为 true 时，进行强类型检查，会报错

##### package.json

1. 你以为这样就配置就够了吗，跑起来会报 Cannot find module 'typescript'，你还没装typescript呢

   ```
   yarn add typescript -D // 参考了好几个都是放在了开发环境emmm,想不好
   ```

------

### 开始踩坑

##### 替换后缀

1. 先来把文件名后缀替换一下吧，把.js 改成了 .ts，扎铁了，直接红线 "找不到名称“div”", "运算符“>”不能应用于类型“string”和“Card”"???。
2. 看了一下别人的文件，后缀是.tsx，那我也改吧，居然真的可以
3. 最后是在文档JSX这里找到一句话，想要使用JSX，必须做两件事，1. 给文件一个.tsx扩展名。那也就是说如果是纯js，那么直接后缀改成.ts就好了，如果是JSX，那就要改成.tsx。

##### 类型定义文档

1. 组件修改run成功，hin开心，好，准备全部替换，先开始入口的main.js

2. 改后缀...好了，刚改完后缀就已经红线了，

   “无法找到模块“react-dom”的声明文件”

   “Try `npm install @types/react-dom` if it exists or add a new declaration (.d.ts) file containing `declare module 'react-dom';`”

3. 嗯，没有安装类型定义文件，错误原因是由于 `React` 和 `React-dom` 并不是使用 TS 进行开发的，所以 TS 不知道 `React`、 `React-dom` 的类型，以及该模块导出了什么，比较幸运的是在社区中已经发布了这些常用模块的声明文件 [DefinitelyTyped](http://definitelytyped.org/) 。

4. 但毕竟是常用的有，不常用的还是要自己声明，我还不会，头疼

5. 而且这里巨坑的是，人家都是react,react-dom等等都没有，我的@types下面一开始居然就有react的声明文件，也不知道是哪里的依赖一并下载了，搞得我还纠结了很久为什么我没装也跑成功了

6. 装一下

   ```
   yarn add @types/react-dom -D
   yarn add @types/react-redux -D
   
   // 后面还有react-router-dom， react-router-redux， lodash等都要装
   // 反正第三方的只要不是typescript写的就要给声明，他也会给提示
   // 然后现在巨坑的是mirror-create的声明没有，我又不会写QAQ
   ```

##### 类型检查

1. 装好了之后发现还是有红线。(这里把router，store先注释了，因为会涉及到很多地方的更改，先只是拿之前改好的那个组件用)

2. “参数“Component”隐式具有“any”类型。”，想了下应该是没给类型注释，当然事实也确实如此

3. 与JavaScript的弱类型不同，typescript的一大重点就在于静态类型检查，所有的变量都要给他一个对应的类型。

4. 具体类型这个我还没看完，所以不放在这里吧，这个是可以细讲很多的，

   基本的boolean, string, number,

   数组有点特别，元素类型[] 或者Array<元素类型> 两种写法

   然后万能的any，呵呵呵呵呵不知道是什么就写any，any用时一时爽，一直用一直爽

##### antd

1. antd要升级到最新版本，不然有些组件会报tagName missing，别问我为什么知道，自闭

------

### TypeScript - React

1. 当我们使用TypeScript定义一个新的React组件的时候，我们必须申明接口的属性和状态
2. // 大概意思就是说，要有<Props, State> // 然后这两个名字你可以自己起
3. 但是如果只是一个render渲染，不需要props,也不用state，应该，可以不写<any,any>这样

##### 接口

1. 我真的有在想，这里的接口是不是可以对应成JavaScript里的object

2. 来写一个PageProps接口

   ```
   interface PageProps{
   	name: string;
       age: number;
   }
   interface: PageState{
       visible: boolean;
   }
   ```

##### 类

1. 先是一个无状态的

   ```
   export default class List extends React.Component < PageProps, any > {
   	constructor(props: PageProps){
           super(props);
   	}
   	render(){ ... }
   }
   ```

2. 再来个有状态的

   ```
   export default class List extends React.Component < PageProps, PageState > {
   	constructor(props: PageProps){
           super(props);
           this.state = {
               visible: false
           }
   	}
   	render(){ ... }
   }
   ```

3. 如果没有props,我猜可以这样

   ```
   export default class List extends React.Component < any, PageState > {
   	state: PageState = {
       	visible: false
       }
   	render(){ ... }
   }
   ```

   