关于redux

目录：

1. 文件结构

2. 配置（依赖以及插件）

   (1) redux-thunk

   (2) mirror-creator

   (3) redux-devtools-extension

3. action

   (1) actionType.js

   (2) action.js

4. reducer

5. store

   (1) createStore

   (2) combineReducers

   (3) applyMiddleWare

   (4) bindActionCreators

6. store与视图层的绑定

   (1) Provider

   (2) connect

7. 

尚未验证：

1. action中的async，和使用api
2. store中的路由中间件

项目地址：

​	<https://github.com/nankaNULL/typescript-react-webpack>

正式开始：

------

### 文件结构

1. 这里还是参考了一下长卿的把store给单独拿了出来，将action, actionType, reducer放在store文件夹下面，但放的是全局所需的，局部的reducer还是放在对应的文件下面

2. ```
   store ( // 文件结构 )
   	index.js
   	actionType.js
   	action.js
   	reducer.js
   ```

------

### 配置（依赖以及插件）

##### 安装

1. ```
   yarn add redux, react-redux, react-router-redux, redux-thunk, mirror-creator
   
   yarn add redux-devtools-extension -D
   ```

##### redux-thunk：

1. 前面的不怎么用说，主要是从后面几个插件开始说起，先是redux-thunk

2. redux-thunk中间件可以让`action`创建函数先不返回一个`action`对象，而是返回一个函数，函数传递两个参数`(dispatch,getState)`,在函数体内进行业务逻辑的封装

   然后很常见的写法就是

   ```
   function addIfOdd() {
   	return (dispatch, getState) => {
           const currentValue = getState();
           if (currentValue % 2 == 0) {
           	return false;
           }
           //分发一个任务
           dispatch({})
       }
   }    
   ```

3. 注入自定义参数的方法 // 参考npm文档

   ```
   // api
   thunk.withExtraArgument(api)
   
   // use
   const store = createStore(
     reducer,
     applyMiddleware(thunk.withExtraArgument(api))
   );
   
   //later
   function fetchUser(id) {
     return (dispatch, getState, api) => {
       // you can use api here
     }
   }
   ```

##### mirror-creator：

1. npm文档中对这个包的解释是： One more way to create an object with values equal to its key names.

   我的理解是，根据键名然后进行一个映射，方便起名字（捂脸

2. 然后为了怕重名，可以添加prefix这么个参数在后面

   ```
   import mirror from 'mirror-creator';
   export default mirrorCreator  （[
     ' SOME_ACTION_TYPE '，
   ] ，{  prefix ：' mydomain / ' } ）;
   ```

##### redux-devtools-extension:

1. 这个插件有点意思，之前一直在用却不知道是因为这个在起作用，直接上链接吧，说也说不太清楚

   https://github.com/yellowfrogCN/reduxDevTools/blob/master/README.md

   在以前利用redux-logger打印日志，用了这款插件之后就能很直观的看到结构，但这个是需要到chrome商店里下载redux DevTools配合着一起使用的

2. 使用的话就是把之前的中间件包裹一下，上代码：

   ```
   import { composeWithDevTools } from 'redux-devtools-extension';
   composeWithDevTools(
   	applyMiddleware(...middleware)
   )
   ```

------

### Action

1. 这里这个目录结构emm感觉不是很好，但也没想好应该怎么弄，暂时吧actionType和action的放一起了
2. actionType比较简单先讲这个吧

##### actionType.js

1. 首先是之前讲的mirror-creator这个插件，嗯，给type都对应起个名字

2. 其次是导出这里，这里两边参考了一下最后决定是用export const editorActions = mirror([ ... ]);这样的方式导出，想了想还是觉得不同模块的actionType都需要分开来写，哪怕他真的很少，这样更适合优化，其他的就没什么差了

3. 然后引入的时候按需引入，使用，例如

   ```
   switch (action.type) {
   	case bookAction.BOOK_MENU_LIST: return ...;
   	default: return state;
   }
   ```

##### action.js

1. 这里是参考了长卿的这个，把所有的action写在了一起，和上面的分开写好像矛盾了，但不知道为什么就是觉得这样更方便，

   使用的时候直接 import * as actions from '@/store/action'

2. 除此之外是写法上进行了修改，

   ```
   getAgents = (payload) => async (dispatch, getState,{API}) => {
       ajax.emm().then(res=>{ // 这样可以这样写吗？还没验证过
       	dispatch({type:..., payload})
       })
   }
   ```

3. 这里需要注意的一个是async异步，（但是这一块我还不太清楚）

4. 还有一个是参数这里，这里的参数，是根据middlewares的，他原本dispatch和getState两个参数，第三个参数{API} 来自thunk

   thunk之前有讲过，他有个API能自定义参数 thunk.withExtraArgument(api)，这便是第三个参数的来源

------

### reducer

1. action只是描述了有事情发生，以及要怎么做，但是没有指明应用改如何更新state（白话一点就是，没有返回一个新的state）

2. reducer就是一个函数，接收旧的state和action，返回一个新的state。返回新的state的时候，最好是Object.assign新建一个对象把值拷贝过去（和getDerivedStateFromProps好像啊这个思路）

   这里我看云日志偶尔会用深拷贝

   ```
   // 引入
   import { cloneDeep } from 'lodash';
   // case里的代码
   const clone = cloneDeep(state);
   const { widgets } = clone;
   clone.widgets = [...widgets, action.payload]
   return clone;
   ```

   emm，说实在他这个深拷贝我也不是很理解，Object.assign虽然不是拷贝所有，但他也是拷贝了第一层的，然后他赋值的这个感觉，就有点emm，如果是再深下去我倒是可以理解。（感觉像是两个风格的人在写）

3. 需要注意的是，reducer需要保持纯净，不要进行任何操作（API请求，修改传参等等，这些放到action里做）

4. 然后在文件结构这里的话，store下面，还是放全局的，局部的放在对应的文件夹下（哦，对，一开始好像也有讲），主要是他这里reducer导出的时候，一个文件就一个嘛，但action又要分开写比较好，总不能模块一的和模块二的action写在同一个reducer里然后switch,case做判断吧，state也不一样啊，所以还是分开

5. 然后上一下代码吧

   ```
   const initialState = { // 该模块下的state
       list:[];
   }
   const globalReducer = (state = initialState, action) => { // 更新state
       switch (action.type) {
           case globalAction.GET_LIST: return Object.assign({}, state, {
           		list: action.payload
           	});
           default: return state; 
       }
   }
   export default globalReducer; // 导出reducer
   ```

------

### Store

1. store该怎么说呢，一个仓库？一个商店？

   store对象维持应用的state，提供getState()获取state，提供dispatch()更新state

   redux应用只有一个单一的store对象，所以reducer需要组合

2. store的API有，然后根据API来看吧

   createStore

   combineReducers

   applyMiddleWare

   bindActionCreators

   compose

##### createStore(reducer, [initialState], enhancer)

1. 初始化store,创建一个 Redux 存储，用于存储应用程序的完整状态树

2. 说实在后面这个参数还真没怎么见用过，第三个参数是 功能，增强器，可以选择指定它来增强存储的第三方功能，例如中间件，

   所以常见的是createStore(reducer, thunk)这样的

##### combineReducers(reducers)

1. 开头有说，redux应用只有一个单一的store对象，那多个reducer里面多个state怎么存储呢，就需要用这个组合一下

2. 上代码

   ```
   // 先引入
   import globalReducer from './reducer'; 
   // 所有的reducer的集合
   const appReducer = { 
     globalReducer
   };
   // 初始化store
   const store = createStore(combineReducers(appReducer),thunk);
   ```

   现在是升级版的createStore了

##### applyMiddleWare

1. 同样，也是来自createStore中，第三个参数enhancer，redux中提供关于中间件的增强器，结合一下使用thunk中间件，进行异步操作

   store再次升级createStore(combineReducers(appReducer),applyMiddleWare(thunk))

2. 然后我们的thunk又可以添加自定义参数，就变成了

3. ```
   const middlewares = thunk.withExtraArgument({API});
   const store = createStore(
     combineReducers(appReducer),
     applyMiddleware(middlewares)
   );
   ```

4. 其实到这里还没完，没有添加上路由中间件，但老实说我不知道这个干吗的，但不加会报错，加上之后应该是这样的

   ```
   import createHistory from 'history/createBrowserHistory'; 
   // 这个history，react-router-dom是依赖的，所以不用装，（但为什么我之前要装，，）
   const history = createHistory();
   const middlewares = [thunk.withExtraArgument({API}), routerMiddleware(history)];
   const store = createStore(
     combineReducers({ routing: routerReducer, ...appReducer }),
     composeWithDevTools(applyMiddleware(...middlewares))
   )
   ```

##### bindActionCreators

1. ```js
   bindActionCreators(actionCreators, dispatch)
   ```

2. 参数：

   actionCreators(Function or Object)：一个action creator或者键值是action creators的对象

   dispatch：由Store提供

3. 惟一使用 `bindActionCreators` 的场景是当你需要把 action creator 往下传到一个组件上，却不想让这个组件觉察到 Redux 的存在，而且不希望把 Redux store 或 dispatch 传给它。

------

### Store与视图层的绑定

##### Provider

1. 要把store绑定在视图层上，会用到connect和Provider，我一开始的时候只用connect，写完之后他报错，

   为什么报错呢，

   因为 “你无法使用connect()去connect一个没有继承Provider的组件，也就是说如果你想在某个子组件中使用Redux维护的store数据，它必须是包裹在Provider中并且被connect过的组件，Provider的作用类似于提供一个大容器，将组件和Redux进行关联，在这个基础上，connect再进行store的传递。”（参考api文档）

2. 那么上面说到他是一个大容器，所以Provider写在最外面，是最大的父级，包裹所有子级，通过children进行传递store，

   ```
   <Provider store={store}>
   	<Component />
   </Provider>
   ```

##### connect

1. api文档给出的话是“It does not modify the component class passed to it; instead, it returns a new, connected component class for you to use.”，

   有点像是一个高阶组件，包裹当前的，然后将state,action通过props传递，让子组件可以使用

2. connect有四个参数connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])

   我感觉一般用的比较多的就前两个，state，和action

   (1) 这里state不要绑错，比如我要的是globalReducer下的state，那就写成

   ```
       const mapStateToProps = (state) => ({
           globalReducer: state.globalReducer
       })；
   ```

   (2) action的话

   ```
       const mapDispatchToProps = (dispatch) => ({
          //云日志这里是把需要的action一个个写了近来
       });
       // 中金这个是拿到所有，（反正我之前的action都写在一个文件里的）
       import * as actions from "@/store/action";
       const mapDispatchToProps = (dispatch) => bindActionCreators({...actions}, dispatch)
   ```

3. 用的时候是

   ```
   export default connect(mapStateToProps, mapDispatchToProps)(yourComponent)
   ```

4. 用修饰器的话，没有什么特别的地方，就是要用到babel的插件（babel-plugin-transform-decorators-legacy），然后在babelrc里配置一下，然后直接

   ```
   @connect(
     (state) => state.globalReducer,
     (dispatch) => bindActionCreators({ ...action }, dispatch)
   )
   ```