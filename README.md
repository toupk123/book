# 模块化

模块就是实现特定功能的一组方法。里面都封装好一些function,其他人拿到这个模块，只需要使用就行

```js
function main(){

}

function add(){

}
```

这里面的函数就可以直接调用,但是如果直接放在全局环境的话，就会污染全局环境。所以为了防止不同模块之间看不出直接关系(之间变量不互相影响)。

## 空间变量

为了解决上面的缺点，所以就可以把这个模块放在一个对象里面，不然的模块成员就放在这个对象里面。

```js
var module = new Object({
  _cout:0,
  m2:function(){
    ///
  }
})

```

上面的函数m1()，封装在这个对象里面。使用的时候，只需要调用这个对象的属性。但是这里又存在另一个问题，那就是对象里面的属性，可以被外部随便调用。

```js

module._count = 123;

```

所以这里需要利用到闭包的特性，创建一个即时执行的函数。

```js
var module = (function(){
  var   _cout= 0,
  var m2 = function(){
    ///
  }

  return {
    m2:m2
  }
})()

```

### 放大模式

```js
module2 = (function(mod){
  mod.m1 = function(){
    ///
  }

  return mod

})(module1)

```

## 模块规范

目前主要的规范有两种，一种是commonJS和AMD

common主要是使用在node环境中

AMD是异步加载模块定义，因为在浏览器环境中，模块之间加载速度并不是固定的，所以在使用一个模块之前，必须得保证模块已经加载完毕。

require([module],callback)callback会在加载完模块后回调函数，这样就可以保证模块加载完之后，再使用该模块。

CMD是推崇就近，AMD推崇前置，AMD就是加载完之后才执行回调函数。CMD就是运行到依赖以后，再去加载对应模块，加载完之后，再执行下午的代码。

其实CDM的执行机制，也就是和CMD是一样的，执行到加载模块的时候，才会去加载。

### ES6的模块化

import的机制，也是和AMD类似，加载完成之后，才会执行代码。

但是得注意区别

1. CommonJS模块输出的是一个值的拷贝，也就是返回一个对象或者值的副本，但是es6是输出一个值的引用。
所以值的引用，会导致es6模块中的值如果发生变化，那模块中的变量值也就会发生变化

2. commonJS是运行时加载，es6是编译时就输出接口。
