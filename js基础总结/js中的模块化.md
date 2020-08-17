# js中的各种模块化

js的模块化规范目前总共有四种：

1. ADM
2. CDM
3. CommonJs
4. Es6

## AMD规范

推荐提前依赖，需要依赖的js文件，需要提前书写。AMD模块加载，并不会影响到后面的代码

```js
define(['./a','./b'],function(modulea,moduleb){
  
  
})

// 记载完成a.js 和b.js就会执行模块里面的代码

```

## CMD规范

推荐就近依赖，但实际上已经通过静态分析提前加载js，在代码require时，就已经加载完js，和AMD只是写法不同。

```js
define(function(require,exports,module){
  // 其他代码
  var s = require('./s')
  // 需要时才加载
})

```



## CommonJS

基本用于node环境的规范，其中module表示模块本身，export 模块导出的部分，require加载模块，使用require加载模块时。被加载的模块会立即调用，然后导出模块的返回值。

```js
const cp = require('ss')


export.foo = 'ss';
// 运行时node才会真正加载模块。同时node会将首次加载的模块进行缓存，确保下次不用重新加载模块。
```

## ES Module

因为CommonJS是基于运行时加载模块，所以在浏览器方面时不适用的，因为模块加载需要消耗时间。

1. es Module是规范浏览器环境。esModule是值的引用，不管是什么值
2. common是动态编译，es是静态分析，使用import会被提升至模块最前面，并且import无法动态引用

```js
import {} from "./s.js"


export const ss =123
// 在浏览器中使用es module时，必须在sceipt标签中加入type=module
```



## UMD

通过一层执行函数，兼容AMD/CDM/Common 等模块规范

```js
(function(root,factory){
 	if(typeof exports === 'object'){
    module.exports = factory()
  } else if (typeof define === 'function' && define.amd) {
    define(factory);
  } else {
    this.eventUtil = factory();
  }
})(this,function(){
  	
})

```

