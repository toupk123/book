1. 手写new函数

```js
// function 构造函数

function Sa(x){
    this.x = x
}

function myNew(fn){
    if(typeof fn !=='function'){
        return new Error('fn不是构造函数')
    }
    var obj = {};
    Object.setPrototypeOf(obj,fn.prototype)
    fn.apply(obj,[...arguments].splice(1));

   return typeof obj === 'object'&& obj!==null?obj:{}
}


```

2. 手写call

```js
Function.prototype.mycall= function(context=window){
    context.fn = this;
    let res = [...arguments].slice(1);
    let resFn = context.fn(...res);
    delete context.fn;
    return resFn
}

```

3. 手写一个bind
```js
Function.prototype.bind = function(context=window){
    let res = [...arguments].splice(1);
    return  function(){
        this.call(context,res)
    }
} 
```

4. 手写一个Ajax

```js
const xhr = new XMLHttpRequest();
xhr.open('GET'.'') 
xhr.onreadystatechange = function(){
if(xhr.readyState ===4&&xhr.status===200){
    
}
}

xhr.send()

```

