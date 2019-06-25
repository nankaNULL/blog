ES6 修饰器

参考文档

1. <http://taobaofed.org/blog/2015/11/16/es7-decorator/>

目录

1. 对方法的修饰
2. 其他

------

好久没写了，摸鱼，参考的这个文档写的挺详细的了，引入的比喻也很好，是为了在原有的基础上添加一些功能或者说修饰，让他变得更好。本文档纯粹摸鱼，另外在添加一些自己在看的过程中不太懂的地方

### 对方法的修饰

代码来着参考文档，文档给的例子是创建一个类，用来创建钢铁侠，并初始化他的攻击力，防御力，血量这些数值

1. 首先得有这个类让我们去修饰

   ```
   class Man{
     constructor(def = 2,atk = 3,hp = 3){
       this.init(def,atk,hp);
     }
   
     init(def,atk,hp){
       this.def = def; // 防御值
       this.atk = atk;  // 攻击力
       this.hp = hp;  // 血量
     }
     toString(){
       return `防御力:${this.def},攻击力:${this.atk},血量:${this.hp}`;
     }
   }
   
   var tony = new Man();
   
   console.log(`当前状态 ===> ${tony}`); 
   ```

2. 接下来我们打算对他的初始值做修饰了，通过装备增强他的防御力

   ```
   function decorateArmour(target, key, descriptor) {
     const method = descriptor.value;
     let moreDef = 100;
     let ret;
     descriptor.value = (...args)=>{
       args[0] += moreDef;
       ret = method.apply(target, args);
       return ret;
     }
     return descriptor;
   }
   ```

   这里问题就来了，首先为什么函数带了三个参数，...args为什么能拿到一个数组，为什么要method.apply(target, args) 

   下面对问题进行解答

   (1) 为什么函数可以带三个参数

   1. 修饰器本质上是利用了Object.defineProperty属性

      语法： Object.defineProperty(obj, prop, descriptor)

      obj 是要在其上定义属性的对象

      prop是需要修改的属性的名称

      descriptor是属性描述符，包含（enumerable，configurable，writable，value）默认false

   2. 显示调用

      ```
      // 显式
      Object.defineProperty(obj, "key", {
        enumerable: false,
        configurable: false,
        writable: false,
        value: "static"
      });
      ```

   3. 对应到这段代码中

      ```
      function decorateArmour(target, key, descriptor) {
        // target 是 Man类
        // key 是修饰的init方法名称
        // descriptor 是init方法的属性描述符
        const method = descriptor.value;
        // descriptor.value 拿到的其实就是init方法
        // console一下会发现结果为 function init(def, atk, hp) { ... }
      }
      ```

   (2) (...args) 这个又是什么

   1. ES6 引入 rest 参数（形式为`...变量名`），用于获取函数的多余参数，这样就不需要使用`arguments`对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中

      ([http://es6.ruanyifeng.com/#docs/function#rest-%E5%8F%82%E6%95%B0](http://es6.ruanyifeng.com/#docs/function#rest-参数))

   2. 上面说过descriptor.value其实就是拿到init方法，而后对该方法进行改写

      ```
      // 原本
      descriptor.value = (def, atk, hp)=>{}
      // 现在
      descriptor.value = (...args)=>{
        // args 其实就是def, atk, hp
        console.log(args) // [2,3,3]
        args[0] += moreDef; // 就是在原本def的基础上加moreDef
      }
      ```

   (3) method.apply()是什么情况

   1. 回顾一下Function.prototype.apply() 调用方法并为他指定一个this对象

      ```
      const method = descriptor.value;
      descriptor.value = (...args)=>{
        args[0] += moreDef;
        ret = method.apply(target, args);
        // 所以这里就是再次调用init的方法，并且把已经修改过的值传过去
        return ret;
      }
      ```

3. 最后添上修饰

   ```
   class Man{
     ...
     @decorateArmour
     init(def,atk,hp){
       ...
     }
     ...
   }
   ```

### 其他

1. 这里不详细写了，

2. 修饰器还可以叠加使用，另外也能对类进行修饰

3. 修饰器也可以添加参数，但这需要再包裹一层

   ```
   function addFly(canFly){
     return function(target){
       ...
       return target;
     }
   }
   ```

   

