# 关于ts的一些基础知识

​	ts可以看作是对js弱型语言的强化版本，本质还是js语法，只是补充了一些类型限制，并且这种限制只是在开发期，在js运行期间没有办法进行限制。

## 基础类型

```js
var num:number = 1;
// num = '2';声明之后，直接复制不同类型， ts会进行报错
var str:string = '2';
var boo:boolean = true;
var arr:Array<stirng> = ['12']; // or var arr:string[] = ['12'];
```

## 元组

```js
let x :[string,number] = ['hello',12]; 
x = [12,'hello'] //error 
x[0].substr(1)// error
```

这种形式固定了数组里面每个元素的类型，所以这种元素每个元素类型不一样，可以调用的方法也不一样

## 枚举

```js
enum color {
  red=12,green=13,blue=14
}
let c:number = color.red
```

## void 和any

