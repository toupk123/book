# 手写js代码(待续)

## 手写new
```js
function New(func){
    var res = {}
    if(func.prototype !== null){
        Object.setPrototypeOf(res,func.prototype);
    }
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));

    if((typeof ret === 'object' || typeof ret ==='function')&& typeof !== 'null'){
        return ret 
    }

    return res
}

```

## 手写call || apply
```js
Function.prototype.call2 = function (context=window){
    context.fn = this;
    let res = [...arguments].slice(1);
    let resFn = context.fn(...res);
    delete context.fn;
    return resFn
}

// apply 
Function.prototype.apply2 = function(context=window){
    context.fn = this;
    let resFn = context.fn(...arguments[1])
        delete context.fn;
    return resFn
}

```

## bind手写
```js
// 这里的难点就bind包裹一层后，还得需要考虑new函数对于this指向的影响；
Function.prototype.bind2 = function(context){
    var slef = this;
    var args = [...arguments].slice(1)

    var funs =  function (){
       return  self.apply(  this instanceof funs ? this : context,[...args,...[...arguments]])
    }
    funs.prototype = Object.create(this.prototype)            
    /*var fNOP = function () {};
    fNOP.prototype =  this.prototype
    funs.prototype = new fNOP();*/
    return funs;
}

```


## promise手写

```js
const PENDING = 'pending';
const FULFILLED = 'fuldilled';
const REJECTED = "rejected";

function Promise(constructor){
    let that = this;
    that.status = PENDING;
    that.value = undefined;
    that.reason = undefined;
    that.onFulfilledCallbacks = [];
    that.onRejectedCallbacks = [];

    function resolve(value){
        if(that.status === PENDING){
            that.status = FULFILLED;
            that.value = value;
            that.onFulfilledCallbacks.map(fn=>fn())
        }
    
    }
    function rejected(value){
        if(that.status === PENDIG){
            that.status = REJECTED;
            that.reason = value;
            that.onRejectedCallbacks.map(fn=>fn())
        }
    }
    try{
        constructor(resolve,rejected)
    } catch(e){
        rejected()
    }
}

Promise.prototype.then = function (onFulfilled,onRejected){
      onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    // onRejected如果不是函数，就忽略onRejected，直接扔出错误
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
     let promise2 = new Promise((resolve, reject) => {
            if(this.status === PENDING){
            this.onFulfilledCallbacks.push(()=>{
                setTimeout(() => {
                try {
                    let x = onFulfilled(this.value)
                    resolvePromise(promise2, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
                }, 0);
            })
            this.onRejectCallbacks.push(()=>{
                setTimeout(() => {
                    try {
                    let x = onRejected(this.reason)
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                }, 0);
            })
        }

        if(this.status === FULFILLED){
            setTimeout(() => {
            try {
                let x = onFulfilled(this.value)
                resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
                reject(e);
            }
            }, 0);
        }

        if(this.status === REJECTED){
            setTimeout(() => {
                try {
                let x = onRejected(this.reason)
                resolvePromise(promise2, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            }, 0);
        }
    }
    return  promise2
}   


function resolvePromise(){
  if(x === promise2){
    // reject报错
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
    let called;
  // x不是null 且x是对象或者函数
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      // A+规定，声明then = x的then方法
      let then = x.then;
      // 如果then是函数，就默认是promise了
      if (typeof then === 'function') { 
        // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
        then.call(x, y => {
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          // resolve的结果依旧是promise 那就继续解析
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          // 成功和失败只能调用一个
          if (called) return;
          called = true;
          reject(err);// 失败了就失败了
        })
      } else {
        resolve(x); // 直接成功即可
      }
    } catch (e) {
      // 也属于失败
      if (called) return;
      called = true;
      // 取then出错了那就不要在继续执行了
      reject(e); 
    }
  } else {
    resolve(x);
  }



}
```

## 手写insatceOf
```js
function insantceOf(obj,fn){
    var objPrototype = obj.getPrototype();
    var fnPrototype = fn.prototype;
    whlie(true){
        if(objPrototype === null) return false
        if(objPrototype === prototype) return true
        objPrototype = objPrototype.getPrototype();
    }
}

```


## 继承


```js

function SpuerClass(name){
    this.name = name;
}

SpuerClass.prototype.setName = function (name){
    this.name = name
}

function SubClass(name,color){
    SpuerClass.call(this,name);
    this.color = color;
}

SubClass.prototype = Object.create(SpuerClass.prototype);
SubClass.prototype.constructor = SubClass
```


