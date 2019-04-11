参考博客 https://github.com/YvetteLau/Blog/issues/6

本文所有代码均参考上面这个博客

目录结构：

1. this的绑定规则

   (1) 默认绑定

   (2) 隐式绑定

   (3) 显示绑定

   (4) new绑定

2. this指向丢失问题

   (1) 隐式绑定中this丢失

   (2) 回调函数中的this丢失

   (3) 显示绑定中的this丢失

3. 箭头函数下this的指向问题

4. 总结



正文

关于this指向性问题，其实感觉理解的还不是特别透彻，这里算作是整理吧，不然我要睡着了。

感觉好像也写不出什么大纲，先写着在看吧

------

### this的绑定规则

1. this的绑定规则，按优先级从高到低进行排序，new绑定 > 显示绑定 > 隐式绑定 > 默认绑定
2. 接下来对每种绑定进行分析，从最简单的开始吧

##### 默认绑定

1. 如何判断什么时候是默认绑定：

   (1) 默认绑定的优先级最低，所以，当其他都不符合的时候，就使用默认绑定了

   (2) apply,call 传入的是null, undefined的时候，用默认绑定

2.  默认绑定时候的指向是什么：

   (1) 默认绑定下，this指向全局对象（但这是非严格模式下的，严格模式下会报错）

   (2) 浏览器环境运行，指向window; node环境中运行，就是undefined

3. 贴个代码，直接拿参考博客的上

   ```
   function sayHi(){
       console.log('Hello,', this.name); // Hello, YvetteLau
   }
   var name = 'YvetteLau';
   sayHi();
   ```

##### 隐式绑定

1. 隐式绑定，看上下文，看离他最近的那一个，其实感觉，默认绑定也可以算是隐式绑定的，

   因为他看的是上下文嘛，，这个还是待会下面分析看看吧，我也是突发奇想

2. 基本上都是，函数的调用在某个对象上触发，这样，上代码分析

   ```
   var name = 'Wiliam';
   var person = {
       name: 'YvetteLau',
       sayHi: function sayHi(){
           console.log('Hello,', this.name); // Hello, YvetteLau
       }
   }
   person.sayHi();
   ```

   你看这代码，person下的sayHi函数，然后去找上下文，person下的name对吧

3. 来多一点的，对象属性链的

   ```
   var person2 = {
       name: 'Christina',
       sayHi: function sayHi(){
           console.log('Hello,', this.name); // Hello, Christina
       }
   };
   var person1 = {
       name: 'YvetteLau',
       friend: person2
   };
   person1.friend.sayHi();
   ```

   person1下的friend，是person2，person2下的sayHi函数，然后上下文，person2下的name

4. 但有时会被坑，就是他有时其实是默认绑定，写在后面吧。

##### 显示绑定

1. 显示就非常舒服了呀，都告诉你指向谁了，根本不用你去判断。

2. 他通过call,apply,bind的方式，显式的指定this所指向的对象

3. 就是有一点，.call( null / undefined ) 的话，是用默认绑定

4. 上个代码

   ```
   var person = {
       name: 'YvetteLau',
       sayHi: function sayHi(){
           console.log('Hello,', this.name);
       }
   }
   var name = 'Wiliam';
   var Hi = person.sayHi;
   Hi.call(person); //Hi.apply(person)
   ```

5. 这里如果直接是Hi(); 输出是wiliam，这个this指向丢失下面统一讲。

6. 用了call指向person，就只要找person下的name就好

7. 显示绑定也有可能会this指向丢失，同上，下面讲。

##### new绑定

1. new创建一个新的对象，this就指向这个对象，没什么好讲的，直接代码吧

   ```
   function sayHi(name){
       this.name = name;
   	
   }
   var Hi = new sayHi('Yevtte');
   console.log('Hello,', Hi.name);
   ```

------

### this指向丢失问题

##### 隐式绑定中this丢失

1. 看一个代码

   ```
   var person = {
       name: 'YvetteLau',
       sayHi: function sayHi(){
           console.log('Hello,', this.name);
       }
   }
   var name = 'Wiliam';
   var Hi = person.sayHi;
   Hi();
   ```

2. 这个时候的this指的是window，而不是person了，我是这么理解的，这时候的Hi是被sayHi函数赋值了，所以现在代码可以写成

   ```
   var name = 'Wiliam';
   var Hi = function sayHi(){
   	console.log('Hello,', this.name);
   }
   Hi();
   ```

3. 然后再去调用Hi();这个时候一看就是默认绑定吧，

4. 我之前不是有说默认绑定也可以看成隐式嘛，因为你看他的上下文，离他最近的也确实是window下的name啊

##### 回调函数中的this丢失

1. 回调函数的这个该怎么讲哦，让我想想，先上代码总是没错的

   ```
   var person1 = {
       name: 'YvetteLau',
       sayHi: function(){
           setTimeout(function(){
               console.log('Hello,',this.name);
           })
       }
   }
   var person2 = {
       name: 'Christina',
       sayHi: function sayHi(){
           console.log('Hello,', this.name);
       }
   }
   var name='Wiliam';
   person1.sayHi();
   setTimeout(person2.sayHi,100);
   setTimeout(function(){
       person2.sayHi();
   },200);
   ```

2. 第一个person1.sayHi(); 执行完后其实是剩下的是setTimeout(function(){ ... })，然后又没有设置时间，所以直接执行function(){ ... }，这个时候是默认绑定了

   但是这里哦，如果setTimeout后面是箭头函数，指向就是person1了，这个我不知道该怎么说，因为箭头函数会绑定上一个？？所以已经绑了，就不是默认了？？？

3. 第二个其实可以看成

   ```
   setTimeout(function sayHi(){
   	console.log('Hello,', this.name);
   },100);
   ```

   那就和第一个没有区别了

4. 第三个是，200ms后执行person2.sayHi();所以是绑在person2上的

5. 

##### 显示绑定中的this丢失

1. 老规矩

   ```
   var person = {
       name: 'YvetteLau',
       sayHi: function sayHi(){
           console.log('Hello,', this.name);
       }
   }
   var name = 'Wiliam';
   var Hi = function(fn) {
       fn();
   }
   Hi.call(person, person.sayHi); 
   ```

2. 这里把person.sayHi做参数传入，简化，变成，

   ```
   var person = {}
   var name = 'Wiliam';
   var Hi = function() {
       function sayHi(){
           console.log('Hello,', this.name);
       }
       sayHi();
   }
   Hi.call(person); 
   ```

3. 这样就不是很难懂了，sayHi()已经在Hi函数内部就已经执行了，啊，这个时候好像不能看上下文诶，嗯，果然还是不能混为一谈啊，只能说这个时候不是XXX.fn()吧，所以不是隐式绑定，是默认了

------

### 箭头函数下this的指向问题

1. 这个总结成一句话就是，函数体内的this对象，继承的是外层代码块的this。（这句话真的总结的很棒啊，好几个地方改了一下都可以去套用这句话！！！）通俗一点的将就是向上翻一层
2. 代码不上了。

------

### 总结

1. 也来个总结吧。
2. new绑定 > 显示 > 隐式 > 默认
3. 隐式 XXX.fn() 去判断，剩下就默认
4. 要注意this丢失的问题，简化到最后看看是不是变成了fn() 这样
5. 箭头函数的总之一句话，继承的是外层代码块的this。万事大吉。