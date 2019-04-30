关于api接口的封装

其实在想是写成fetch的封装好还是api接口的封装好，，诶，大家懂就好，这个概念的东西真不知道该怎么讲

目录：

1. fetch

   (1) fetch请求写法

   (2) post 的写法

   (3) 异常处理

2. 对fetch进行封装(目前只有get, pose)

   (1) 自定义http对象

   (2) 将所有请求进行整合

文档参考：

1. https://www.cnblogs.com/libin-1/p/6853677.html
2. https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch

项目地址：

​	<https://github.com/nankaNULL/typescript-react-webpack>

------

### fetch

1. 发送或获取数据，会用到AJAX，然后要写很长一串，还不好记，fetch的话就方便很多。
2. 说到请求的话就会想到ajax（原生的，不是jquery封装的）, axios, 然后现在的fetch，三者具体的区别，用我之前看到的一句话来解释，axios是对ajax的封装，fetch是一种浏览器原生实现的请求方式，跟ajax对等
3. 

##### 基本的fetch请求

1. 写一个最简单的fetch请求（默认get请求）

   ```
   fetch('https://api.github.com/users/chriscoyier/repos')
   .then(function(response) {
   	return response;
   })
   ```

2. 这里的返回结果并不是想象中的，直接拿到数据了，他返回的是个Promise，而真正的数据存储在body中

3. 想要拿到body中的数据还要进行一步转换，这里需要根据返回的响应格式进行对应的转换。

   如果返回的响应是JSON格式的，就调用`response.json`方法来转换数据。

   如果请求一个XML格式文件，则调用`response.text`。

   如果请求图片，使用`response.blob`方法。等

4. 所以现在的代码可以升级为 

   ```
   return response.json();
   ```

   这时再console一下，就可以看到是我们想要的数据了

   

##### 支持的请求参数

1. fetch(input, [init])

   (1) input可以是一个包含获取资源的URL的字符串，也可以是一个request对象

   (2) init配置对象（写成options更好理解吧小声bb），可选参数有method, header, body, mode等，（可参看https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/fetch），这里就讲主要用的那几个

2. method: 请求使用的方法，GET，POST，默认get可以不用写

3. headers: 请求头，是个对象

4. body: 请求的body信息，但GET，HEAD方法的请求不能包含body信息

5. 然后写一个post请求

   ```
   let content = {some: 'content'};
   fetch('some-url', {
     method: 'post',
     headers: {
       'Content-Type': 'application/json' // 1
     },
     body: JSON.stringify(content) //2
   }).then(...)
   ```

   这里需要注意两点，

   一是请求头，我这边发送的是JSON格式的，那么Content-Type就设置成application/json

   二是body这里，还是因为我是JSON格式的，所以body要JSON.stringify(), 将原本的转换为JSON对象（类对象的字符串）

##### 异常处理

1. fetch一开始返回的response，是个promise，数据是这样的

   ```
   body: (...)
   bodyUsed: true
   headers: Headers {}
   ok: true
   redirected: false
   status: 200
   statusText: "OK"
   type: "cors"
   url: "https://api.github.com/users/chriscoyier/repos?"
   ```

   主要是看一个ok（请求成功为true，反之false）, status（状态码）, statusText，当请求不成功，我们就可以抛出一个异常，然后进行一些异常处理，那这里结构也要修改一下

2. 首先是response里进行一个判断，response.ok === false 那就 throw response.status

   然后在catch里面对不同的情况进行处理

3. 上代码：

   ```
   fetch('https://api.github.com/users/chriscoyier/repos')
   .then(response => {
   	if (response.ok) {
           return response.json();
   	} else {
           throw response.status;
   	}
   })
   .then(data => {
       return data; // 成功的返回data，不要以为response.json()就完了
   })
   .catch(status => {
       if (status === 401) {
           window.location.href = '/home';
       }
       else if (status === 404) { ... }
   })
   ```

4. 中金和云日志的对比了一下，感觉长卿这个他是直接在第一个then里做了判断，也没有抛出异常什么的，他那个异常就没用到过的感觉，云日志就和上面这个写法差不多，中规中矩的

其实写到这里fetch也已经差不多了，就是如果then里的操作太烦了可以把里面的function单独拿出来，嗯。

------

### 对fetch进行封装

##### 自定义http对象

1. 这里用new 创建一个自定义对象http，然后构造方法get, post等，对请求进行封装

2. emm具体也不知道该怎么讲，用代码吧

   ```
   class Http{
       request(url, options) {
           // 我这里就简写了，反正就是把fetch返回
           return fetch(url, options)  
       }
       get(url, params) {
           const newUrl = ... ;// 字符串拼接一下，get的请求体
           const options = {method: 'GET' };
           return request(newUrl, options) // 这里拿到返回的fetch，再返回上去
       }
       // post的就省略吧，就是options中多了个headers和body,body要注意转换格式
   }
   export new Http();
   ```

   

##### 将所有请求进行整合

1. 这个时候其实已经可以直接在组件里使用了，

   ```
   import http from './http'
   ...{
       ...
       http.get('http://...',{...}).then(response => {...})
       ...
   }
   ```

2. 但是这样还是不够简洁，我们希望在组件里不要出现url，直接传参拿到response就好，于是在api文件夹下又进行了一次整合（我感觉这次不能叫封装）

3. 整合成一个对象，

   ```
   /* api/index.js */
   export default{
       getTheData(params){
           return http.get('http://...',params)
       }
   }
   
   /* component */
   import API from '@/api';
   ...{
       ...
       API.getTheData({...}).then(response => {...}) // 这个写法太亲切了，感人
       ...
   }
   ```

4. 其实这样已经差不多了，之后是参考长卿的那个，在提取了一下，把method，url单独提出来存在一个文件下，然后同样的还是建立一个对象，对所有请求进行一个整合，只是这次整合不是手动，而是调用函数一次性完成，不写了。