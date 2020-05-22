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



# js中的数据类型





