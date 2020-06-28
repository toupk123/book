# js调用堆栈

从一个简单的赋值操作说起

```js
var s = 12;
```

这里的一行代码操作实际上是分为两步

1. 声明操作
2. 赋值操作

分为两步其实也是因为js是一门解释语言，在代码实际开始运行前，会进行一步预编译，生成AST语法树之后，才会真正进行赋值的操作。

## 执行上下文和执行栈

执行上下文可以被理解为js代码被解释和真正执行的地方，也就是上面代码真正执行的地方。

### 执行上下文种类

1. 全局执行上下文：在当前环境中唯一，比如在浏览器中就是window对象。
2. 函数中的执行环境，存在无数个，每当一个函数执行时，都会创建一个函数执行上下文。
3. Eval函数上下文：指运行在eval函数中的代码，不建议使用。

补充:eval不推荐的原因可能有多个方面

1. 安全问题。传入的字符串代码都可以被执行，那么就很容易受到xss攻击
2. 其迷惑作用域。即使实在函数内部运行eval函数，其内部实际也是指向全局环境

### 执行栈

后进先出结构，用来储存在代码执行期间创建的所有执行上下文。

在首次加载js代码时，会在执行栈底层放入一个全局执行上下文，这也就是为什么我们可以直接调用window对象上属性的原因

每次调用一个函数时，都会向执行栈中放入一个函数执行上下文，当该函数执行文执行完毕后，会pop出来，调用权会交给他下面的执行环境。

### 执行上下文创建

执行上下文分为两个阶段

1. 创建阶段
2. 执行阶段

#### 创建阶段

1. 确定this值，也就是This Binding。
2. **LexicalEnvironment（词法环境）** 组件被创建。
3. **VariableEnvironment（变量环境）** 组件被创建。

```js
ExecutionContext = {  
  ThisBinding = <this value>,     // 确定this 
  LexicalEnvironment = { ... },   // 词法环境
  VariableEnvironment = { ... },  // 变量环境
}
```

##### 词法环境

词法环境又分为两个部分

1. 环境记录：储存变量和函数声明的实际位置
2. 对外部环境的引用：也就是访问其外部的词法环境

词法环境也分为全局词法环境和函数词法环境

1. 全局词法环境，当然也就是没有外部环境的引用。同时环境记录里面储存着全局对象，以及用户自定义的全局变量
2. 函数环境.储存着arguments对象(类数组),自定义在该环境中的变量，对外部词法环境的引用

##### 变量环境

变量环境其实也就是一个词法环境，他具备上述所有的属性。区别在使用ES6中声明的变量会储存在变量环境中。



#### 变量提升的原因

也就是因为在运行代码前，会先分析其环境内的声明变量和函数声明，也就是执行上下文创建阶段。

1. 对于var声明，在词法环境中会被直接赋值为undefined。
2. 变量环境中的let则只会被预先设定为无法使用的变量
3. 同时函数会被提到最前，放在词法环境中，这就是为什么还没有声明函数的代码，但是已经可以提前使用

## 执行过程

回到上面那行代码的过程

1. 首先创建一个全局上下文放入执行栈底，创建执行上下文，同时词法环境中会把变量s赋值为undef
2. 创建完成之后才会执行该代码，将2赋值s。

# js中的垃圾回收

js是自己垃圾收集机制，每隔一段时间就会执行一次释放操作。

- **局部变量**：局部作用域中，当函数执行完毕，局部变量也就没有存在的必要了，因此垃圾收集器很容易做出判断并回收。局部变量中基本类型是存在栈内存，引用类型是存在堆内存。
- **全局变量**：全局变量什么时候需要自动释放内存空间则很难判断，所以在开发中尽量**避免**使用全局变量。同时这类变量不管是基础类型还是引用类型，其值是存在堆内存中。
- **被捕获变量**：也就是闭包内的变量，这类变量都是存在堆内存中。 

以Google的V8引擎为例，V8引擎中所有的JS对象都是通过**堆**来进行内存分配的

- **初始分配**：当声明变量并赋值时，V8引擎就会在堆内存中分配给这个变量。
- **继续申请**：当已申请的内存不足以存储这个变量时，V8引擎就会继续申请内存，直到堆的大小达到了V8引擎的内存上限为止。

V8引擎对堆内存中的JS对象进行**分代管理**

- **新生代**：存活周期较短的JS对象，如临时变量、字符串等。
- **老生代**：经过多次垃圾回收仍然存活，存活周期较长的对象，如主控制器、服务器对象等。



## 垃圾回收算法

### 引用计数

在以前低版本浏览器中使用的方式，这种方式就是查询每个值被引用的次数，一旦次数为0，就将空间释放。缺点就是一旦出现相互引用时，就会先出内存溢出的bug。

### 标记清除

标记清除算法将“不再使用的对象”定义为“**无法到达的对象**”。即从根部（在JS中就是全局对象）出发定时扫描内存中的对象，凡是能从根部到达的对象，**保留并打上标记**。那些从根部出发无法触及到的对象被标记为**不再使用**，稍后会在清除阶段进行回收。

缺点：造成内存碎片化，v8引擎对这方面的处理，是直接把存活的对象往一段靠拢。

# 浅拷贝和深拷贝

##  JSON.parse 和 JSON.stringify

但是会有以下几个问题

1. 会忽视undefined
2. 会忽视symbol
3. 不能序列化函数
4. 不能解决循环引用的对象
5. 不能处理正则

# 防抖和节流

## 节流

是指一段内只执行一次操作

```js
function throttle(fn,wait){
  var canRun  = true;
  var timeout = null
var throttled = function(){
    if(!canRun){
        return
    }
   if(canRun){
    timeout =  setTimeout(function(){
       fn.apply(this,[...arguments])
       canRun = true
     } ,wait)
    canRun = false
   }
 }
   // 添加手动取消功能
 throttled.cancl =   function (){
    	clearTimeOut(timeout);
    	canRun = false;
    	timeout = null
	}
    return throttled
}
```



## 防抖

```js
    function debounce(fn) {
      // 4、创建一个标记用来存放定时器的返回值
      let timeout = null;
      return function() {
        // 5、每次当用户点击/输入的时候，把前一个定时器清除
        clearTimeout(timeout);
        // 6、然后创建一个新的 setTimeout，
        // 这样就能保证点击按钮后的 interval 间隔内
        // 如果用户还点击了的话，就不会执行 fn 函数
        timeout = setTimeout(() => {
          fn.call(this, arguments);
        }, 1000);
      };
    }
```

# Promise相关问题

promise在异步编程方面，可以用来解决回调地域，并且进行链式调用。

1. promise有三个状态，pending，fulfilled和rejected。一但状态修改，则无法再改变状态。

2. promise一但新建，就会立即执行无法取消。如果不设置回调函数，Promise内部抛出的错误，在外部就无法捕获原因

   ```js
   try {
     new Promise(function () {
       ss
     })
   } catch (e) {
     console.log(e);
   } 
   console.log('111')
   // 在外部是无法捕获的，但是js引擎不会停止解析，会继续执行后面代码
   // 111 还是会被打印
   new Promise(function () {
       ss
     }).reject(e=>{
         console.log(e)
     }) // 类似于设置了reject并且捕获了错误原因
   ```

   promise如果抛出错误，就会一直向后传递，直到被捕获。

   ```js
   getJSON('/post/1.json').then(function(post) {
     return getJSON(post.commentURL);
   }).then(function(comments) {
     // some code
   }).catch(function(error) {
     // 处理前面三个Promise产生的错误
   });
   ```

3. 一般对于错误的捕获，尽量使用catch方法来捕获，不推荐使用then方法，因为catch还可以捕获到前面

## promise中的一些api

1. finally。不关最后状态如何，finally方法内函数都会被执行，不过finally中是无法知道前面状态的。

2. all。将多个promise包装成一个新的promise。传入的可以不是数组，但一定得有Iterator。只有当所有的promise状态改变为resolve，才会调用then方法中resolve方法。只要有其中某一个promise的状态为reject，就会立马调用all的状态就会马上变为reject，并且调用reject方法，而不会等待其他结果返回。不过其中promise本身是捕获了这个错误，那么就不会触发all中的reject。

   ```js
   const promise1 = new Promise((res,rej)=>{
     setTimeout(function(){
       rej('测试1')
     },1000)
   })
   
   const promise2 = new Promise((res,rej)=>{
     setTimeout(function(){
       res('ceshi2')
     },3000)
   })
   
   
   
   Promise.all([promise1,promise2]).then((res)=>{
     console.log('res',res)
   },rej=>{
     console.log('rej',rej);
   }); // 可以进行测试，在第一个返回的时候，就已经触发了all的rej
   ```

   

3.  race ，就是和all类似，但只要有一个promise状态改变，就会调用then
4. allSettled。对应上面的all，只要全部状态返回，不关结果都会调用then中的resolve。all中是如果有某一个出现错误，就不会去关其他promise返回结果，调用all中的reject。allSettled还是会等待其他结果。

## promise和原生ajax配合

```js
function getJSON(url) {
  return new Promise(function (res, rej) {
    function handler() {
      if (this.readyState == 4 && this.status == 200) {
        res(this)
      } else {
        rej(this)
      }
    }
    xhr.open('get', 'https://jsbin.com/zokezamufa/edit?js,output');

    xhr.responseType = "json";
    xhr.setRequestHeader("Accept", "application/json");
    xhr.onreadystatechange = handler
    xhr.send();
  })
}

getJSON(URL).then

```

## promise的手写代码

```js
class MyPromise {
    constructor(executor) {
        this.value = undefined;
        this.status = 'pending'
        this.onRejected = [] // 储存失败的函数
        this.onResolved = [] // 储存成功的函数
        var resolve = result => {
            if (this.status != 'pending') {
                return
            }
            this.status = "resolved";
            this.value = result;

            this.onResolved.map(fn => {
                fn(result)
            })
        }
        var reject = result => {
            if (this.status != 'pending') {
                return
            }
            this.status = "reject";
            this.value = result;
            this.onRejected.map(fn => fn(result))
        }
        try {
            executor(resolve, reject)
        } catch (e) {
            reject(e)
        }
    }

    then(onResolve, onReject) {
        const promise2 = new MyPromise((resolve, reject) => {

           
            if (typeof onResolve !== 'function') {
                return onResolve = resule => resule
            }
            if (typeof onReject !== 'function') {
                onReject = resule => {
                    return onReject = resule => resule
                }
            }
     
            if (this.status === 'pending') {
                this.onResolved.push(() => {
                    let x = onResolve(this.value);
                    resolvePromise(this, x, resolve, reject)
                })

                this.onRejected.push(() => {
                    let x = onReject(this.value)
                    resolvePromise(this, x, resolve, reject)
                })
            }
            console.log('cesjo',this)
            if (this.status === 'resolved') {
                const x = onResolve(this.value)
                
                resolvePromise(this, x, resolve, reject)
            }
            if (this.status === 'reject') {
                const x = onReject(this.value)
                resolvePromise(this, x, resolve, reject)
            }
        })
        return promise2
    }
    static all(promises) {
        let arr = [];
        let i = 0;
        function processData(index, data) {
            arr[index] = data;
            i++;
            if (i == promises.length) {
                resolve(arr);
            };
        };
        return new Promise((resolve, reject) => {
            for (let i = 0; i < promises.length; i++) {
                promises[i].then(data => {
                    processData(i, data);
                }, reject);
            };
        });
    }
    static race(promises) {
        return new Promise((resolve, reject) => {
            for (let i = 0; i < promises.length; i++) {
                promises[i].then(resolve, reject)
            };
        })
    }
    static resolve(data){
        return new MyPromise(function(res,rej){
            res(data)
        })
    }

}

function resolvePromise(promise2, x, resolve, reject) {

    if (x === promise2) {
        return reject(new TypeError('产生了循环引用'))
    }
    //防止多次调用

    let called = false
    if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
        try {
            let then = x.then
            if (typeof then === 'function') {

                then.call(x, y => {

                    if (called) return
                    called = true

                    resolvePromise(promise2, y, resolve, reject);
                }, err => {
                    if (called) return
                    called = true
                    reject(err);
                })
            } else {
                resolve(x);
            }
        } catch (e) {
            if (called) return
            called = true
            reject(e)
        }
    } else {
        resolve(x)
    }
};
```





## 其他

promise的使用还和其他知识点有关联，后面再记录。

1. 微任务和宏任务
2. async/await
3. Generator函数

# 前端MVC和MVVM

## MVC

MVC是一种设计模式，将应用分为三层

1. 数据(也就是模型m)
2. 展示层(view)
3. 用户交互层(contoller)

![](G:\js基础知识总结\book\js基础总结\image\1645b7386ed2e6d2.png)



### Model层

这一层存放的是和数据相关的操作，包括修改数据等，也包括control调用的回调函数，也都是放在model层。control监听用户操作后，会调用model上的回调函数，修改model里面的数据。

```js
var Model={
    name:'',
    find:function(){
        
    },
    set:function(value){
        // 一些其他业务逻辑的代码
       Model.name = value
    }
}

```

###  control

页面加载的时候，控制器将事件绑定在视图中，然后处理一些回调，和model层进行必要的对接。所以control里面的代码是能少则少，具体的业务代码应该交给model层或者view层。control仅仅只是一个传递消息的作用。

比如在用户点击某个按钮触发事件，这个事件就是control 部分监听，然后调用model里面的回调函数修改。

```
var ToggleView = {
	this.toggleClass:function(){
		this.view.toggleClass('over',e.data) //至于这里的toggle具体方法，是在view层里面。
		Model.set(e.data) // 同理
	}
}
```



### view

给用户展示，并且与用户产生交互。在js中，视图逻辑也就是在处理html上，view的作用就是将HTML和js连接起来，接受控制器的管理。也就是在js中写HTML代码，并且将HTML代码嵌入到html页面中。



```js
// 伪代码

function createModel(){
    // 生成model数据
    var obj = {
        name:''
    };
    
}

function controller(Model){
    changefn:functon(view){
        // 这里是在监听事件然后调用数据
        Model.set('name',e.data);
        view.show(e.data)
    }
}

var V = function View(control){
    var view = $('ss'); //这里的不一定是真实dom节点，也可能是模板,这里就是在创建真实的dom元素，或者是创建一个新的模板；
    this.init=init;
    this.show = show;
    this.render = render;
    this.view = view;
    init:function(){
     	 view.addEventlistener('change',control.changefn.bind(this,this.view))
    },
    show:function(value){
        // 这里可能是生成一些dom元素碎片，或者之类的模板
        var dom = document.createElement('div')
        dom.innerHTML  = value
        this.render(dom)
    },
    render:function(dom){
        // 一些渲染成html 的方法
        this.view.add(dom) // 挂在到真实的dom节点.
    }
}

var model = new createModel;
var control = new controller(model);

var view = new  View(control,Model);
view.init();


```



总体上来说，MVC将一个组件进行了部分解耦，让每个部分专注自己的这一层，但是解耦程度并不够，MVC中会进行大量的DOM操作，这本身就影响性能。

同时一个组件就会对应一个MVC，当项目比较大的时候，就会出现非常多的MVC。并且model里面数据变化，依赖control去通知view进行更新，也就是当组件越来越复杂之后，control里面的代码也会越来越复杂，开发人员必须去创建更多的回调函数。



## MVVM

相当于在MVC框架，因为在视图层中会进行大量的dom操作，这就很影响性能，所以大部分的框架都已经放弃mvc框架

MVVM也就是指Model、ViewModel、View

## MVVM实现机制

数据挟持+ 发布订阅

1. 在老版本的Angular1.x版本是通过脏值检测来处理。
2. 在vue中是通过Object.defineProperty来达到值变化监听，所以vue在IE8以下并不兼容
3. 

### View

View只是进行视图模板，用来定义结构，本身并不处理数据，只是将viewModel的数据渲染出来。

此外和ViewModel能够进行交互，所以还需要做的就是数据绑定的声明，指令的声明，事件绑定的声明。

```js
<p yg-text="message" />
```

可以理解为带有特殊属性的HTML模板。View就是通过使用这种模板语法声明式，将数据渲染进DOM，相对于这MCV中的View层，不需要view关心model上的数据，view只和VM通信（原先view层需要将model上的数据，渲染到html上）。

对于开发者而言，不需要直接对dom元素进行操作，这大大减少了开发量。



### ViewModel

视图模型，起着连接View和Model的作用，同时处理View中的逻辑。视图模型从model中获取数据，也就是vm中的数据，最后View通过模板，渲染成真实数据。

viewModal的底层会对模板上绑定的属性 或者事件进行监听，当VM中的属性发生变化之后，View层也会得到更新。

```js
<div>{{name}}</div>

vm.name = '测试'
```



对于modal而言，当Model发生变化时，View就会自动更新，View变化，Model也就会更新，其中viewModal作为中间的桥梁，对数据进行挟持，达到双向绑定的效果。

```js
var obj = {
    name:'1' //这里就是简单modal ，现在修改name，自然无法起到挟持的作用
}

 var vm  = {
	name:obj.name,
  	age:12
}
// 这里就相当于Observer，只是对其中一个name进行了观察
Object.defineProperty(vm.name,{
    get:function(){
        // 
    },
    set:function(value){
        // 修改是进行操作
        // 比如修改了里面的name,我们进行修改
        // 通过m -> v
        vm.name = value;
        obj.name = value;
        this.render(vm)  // 通过数据进行重新渲染
    }
})
function  render(vm){
    // 这里会有一个复杂的渲染系统，将模板渲染成真实的dom，并且对模板上一些声明的vm属性，或者指令进行转换为真实数据，和真实的监听事件
}

// 同时在view模板中，我们进行了事件的声明
<input v-model="name" :change="setName" />
    
// 通过vm中的render函数会转为真实的监听函数，监听vm上setName函数
   setName = function(value){
    vm.name = value; // 这里就会同步到Model
    vm.render()  // 通过v  -> m
}
// 这样就大致完成了一个简单MVVVM架构


```

同步上述步骤可以发现，View之和VM进行了交互。同时一些具体的dom操作，和数据挟持的操作会交给底层，这大大降低开发工作者工作量。



## Model

一般是进行数据模型的一些获取操作(例如ajax请求等)，或者是处理数据的操作。

```js
var data = {
    val:0
}


```

## 关于react是属于MVC还是MVVM

准确说react目前来看只是属于view层，并不是一整套的开发框架。包括对数据流状态的管理等。为了解决掉react中的这些问题，flux架构也就出现。



# js中的数据类型





# 设计模式

## 单例模式

单例模式是指一个类只有一个对象，即保证只能实例化一次，必须线程池，或者全局缓存对象。

实现的原理其实也是利用到了闭包，用一个属性去储存已经实例化的对象。



```js
function Singleton(name){
    this.name = name;
    this.instance = null;
    this.getInstance = getInstance
}

getInstance = function(name){
    if(this.instance === null){
       this.instance = new Singleton(name)
    }
    return this.instance;
}

var a = Singleton.getInstance('测试')

```

### 透明的单例模式

对比上一个模式，用户需要调用构造函数上的方法，并不能直接使用new函数，这里其实也是通过闭包实现。

```js
var  Singleton = (function(){
    var instance=null;
    return function Singleton(name){
        this.name = name;
        if(instance === null){
            instance = this;
        }
        // 一些其他实例的方法
        return instance
    }
})()
var s= new Singleton('测试')

```



### 使用代理实现的单例模式

修改上面代码

```js
var Singleton = (function(proxyObject){
        var instance=null;
    return function (){
        if(instance === null){
            instance = new proxyObject(...[...arguments]);
        }
        // 一些其他实例的方法
        return instance
    }
})(String)// 在这里传入构造函数
// 通过这种方式到达解耦，这样我们在构件时可以传入任意的构造函数，实现单例模式

```

### js中的单例模式

上面的模式更像是面向对面语言的单例模式，在js中创建一个对象非常简单，不像java得先声明一个class。

单例模式的核心是创建一个唯一的对象，并且提供访问。

```js
var obj = {}; //全局环境下
// 在其他的作用域下
window.obj.name = 'ceshi';
// 这里就达到了一种类似单例模式，因为window下的obj就是个单例
```

类似于window对象下的obj，我们只需要创建一次，然后在其他地方就可以直接使用了。

当然这样会产生全局变量污染，那么我们这个时候就可以使用命名空间,减少全局变量污染

#### 命名空间

```js
var MyApp  = {
  headerObj:{},
   timeObj:{}
};

```

#### 惰性单例模式

惰性单例模式是指在我们需要的时候，才会创建单例。位于上面书写的代码其实就是一种惰性单例。例如Singleton函数，在没有进行new这个操作之前，其中储存的单例对象并不会直接创建。

在js中，一些全局性质的弹窗提示之类的，为了减少首次加载dom的性能。我们可以在他需要首次弹窗提示时，再进行创建DOM元素。

```js
class confirm extends Component{
    // 其他逻辑代码
    state = {
        init:false // 代表是否进行了回调
        show:false
    }
	click(){
        this.state.show = !this.state.show
        this.state.init = true
    }
	render(){
        return this.state.init&&<div></div> // dom元素
    }
}
```

全局状态下，这也就是一种惰性单例模式。



## 策略模式

策略模式的定义是：定义一系列的算法， 把他们封装起来，并且时他们可以相互替换。

这里的相互替换是指输入同样数据，或者同种属性，然后输出的结果也是类似。

比如对不同评分的员工颁发年终奖，不同的等级就有不同的分法，如果为没一种等级都写一个判断的话，就显得非常的臃肿

```js
if(type ==='A'){
    // 计算
} else (type==='B'){
    
} // 依次
var strategies = {
    "S":function(num){
        return num*4
    },
    "A":function(num){
        return num*3
    }
}

var calculate=function(level,num){ // 本身没有计算能力，而是封装进入strategies
    return strategies[level](num)
} // 这里不用进行任何的判断，并且可以随时添加新的等级判断

```

在业务逻辑中，对于表单我们也可以进行同样的处理，对不同的表单字段进行格式内容判断。

## 代理模式

代理模式是为一个对象提供一个类似网关，用来控制外界对该对象的控制。

![1590649607(1)](G:\js基础知识总结\book\js基础总结\image\1590649607(1).png)·

在js中，比如操作addEventListener 添加事件监听时，因为低版本IE中该事件为addEvent，那么这个时候我们就可以设置一个代理对象add，用来添加监听对象。

```js
add = {
	event:function(){
        // 判断版本
        if(type==='IE'){
            // 就对应IEaddEvent
        } else{
           // 否则就使用addEventListener
    }
}

```

对应外部来说，如果没有代理存在，那么每个去操作监听的对象，都需要进行判断，但是现在这个判断交给了代理。

## 迭代器模式

其实就是类似于ES6中的for each方法，迭代器接口将对象或者数组内部抛出来，其中的业务代码就可以分离出来。

```js
// 没有用迭代器
for(var i =0;i<arr.length;i++){
    // 业务代码,这样的话就比较耦合，并且对每个类似的操作，都需要开发者使用for
}

// 修改一下
function each(arr,fn){
 	for(var i =0;i<arr.length;i++){
		fn.call(arr,arr[i],i)
	}
    // for(var i  in arr)
    // 甚至是复杂的对象结构，也就是es6中提供的迭代器接口
    
}
// 这里只是判断了对象，但是我们也可以对象，外部并不需要知道内部结构
each([1,2,3],function(){
    // 业务代码
})


```

这种直接在函数内封装的迭代就是被成为内部迭代，像es6中直接在需要迭代的对象中写迭代器的就是外部迭代。

## 发布订阅模式

```js
var EventBus = (function(){
    var eventList = {
        
    };
    function add(key,fn){
       if(!eventList[key]){
         eventList[key] = [fn]
       } else {
          eventList[key].push(fn)
       }
    }
  function on(key){
    if(eventList[key]&&eventList[key].length){
      eventList[key].map(fn=>{
          fn(...[...arguments].splice(1))
      })
    }
  }
   
    return  {
        add,
        on
    }
})()
```

在js中本身就会运用到大量的订阅发布模式，比如事件监听。

### 一定需要先订阅之后，才能进行发布？

在实际的情况当中，比如购物车，在用户没有登录前，我们也是可以将这些物品放入购物车内，直到用户登录之后，再将购物车内的商品，放入用户的购物车中。

## 命令模式

