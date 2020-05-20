# 数据类型相关知识

js中内置的数据类型有七种
1. number
2. string
3. undefined
4. null
5. object
6. symbol
7. boolean

使用typeof  都时能够查到对应的字符串值

```js
typeof 42 === 'number'  // true
typeof function(){/**/} === 'function' // true
typeof null  === 'object' // true
```

但如上图所示，有两个例外，一个就是function，一个就是null。
由此可以看出来function其实也就是object的一个子集，函数也就是一个可调用的对象。它内部是有一个[[call]]值,该属性值得它可以被调用。

## 关于undefined

undefined实际上是已经声明，但还未进行赋值操作，而不是没有进行声明。

```js
if(DEBUG){
 /**/  // 这里的DEBUG如果没有在其他模块声明值的话，这里浏览器就会直接报错
}

if(typeof DEBUG  !== 'undefined'){
    // 这里就不会因为没有声明而导致,对于没有声明的参数，typeof也会返回undefined
}

```


## object
所有的对象都会含有 
1. constructor

2. hasOwnProperty(paramName) 用于判断实例上(而不是原型上拥有这些属性)

3. isPrototypeof用于检查传入的对象是否是另一个对象的原型。

4. propertyIsEnumerable(propertyName)判断数据能否使用for-in(也就是可枚举还是不可枚举)

5. toLocaleString 返回对象字符串表示，该字符串与执行环境的地区对应。

6. toString():返回对象的字符串表示

7. valueOf():返回对象的字符串 数值 或者布尔值表示，通常与toString()返回值相同


```js
// 

```

## 数组

关于数组声明时的一些注意点，那就是直接声明Array长度时，但没有进行具体赋值时，可能会产生难预料的bug

```js

const arr = Array(14)
arr.length = 14

// or 

arr[100] = 1;
arr.length = 100

```

有时候为了将一些类数组的数据格式，返回一个数组时，可以使用

```js
Araay.prototype.slice.call(arguments)

// or  es6

const arr =Array.form(arguments) 

```

### 检测数组

1. 可以使用instanceof Array 判断，但这种方式并不是安全的，后面会加说明

2. Array.isArray()

### 数组字符串转换

```js
var a = [1,23,4]
a.toString() // 1,23,4 函数会自动去调用对象原型上的toString()，转换为字符串

```
### 栈方法
数组可以表现的和栈一样，这先进后出，js为数组专门提供了push()和pop（）
这种数组顶部移除，末尾添加的形式。

### 队列方法
队列就是表现为后进先出。
这里就是需要配合shift push

### 数组排序

1. reverse() // 翻转数组

2. sort //对数组进行排序
 
 sort接受一个函数，对前后两个元素进行比较，而后进行排序

 返回负数代表 需要交换位置，返回正数，则代表不需要交换位置

 如果不传入参数，sort则会调用toString()，比较大小，进行升序排序

注：都是返回新的数组，不会改变原数组

3. concat

4. slice

5. splice

6. indexOf

## 基本包装类型

Boolean类型的实例重写了valueOf(),不再返回string,重写了toString返回字符串形式的“true”

Number类型

String类型


## 原生函数

常用的原生函数有：
1. String
2. Number
3. Array
4. Object
5. Function
6. RegExo()
7. Date
8. Error
9. Symbol

可以使用这些原生函数当做构造函数使用，但是创建出来的值，可能就不在于基础类型(例:string)

```js
var a = new String('123123');
typeof a // object

Object.prototype.toString.call(a) // "[object Stirng]"

```
这也就解释了，基本类型如string本身是没有 length 之类的属性或者方法，但却能够直接调用，根本原因还是js在执行的时候，将这些基本数据类型，先给转换为封装对象String

如果需要获得封装的返回基本类型的值，可以直接调用 valueOf

## 值类型转换
抽象操作ToString负责进行类型转换。对于普通对象来说，除非自行对象，否则都是toString()返回内部属性[[class]]

数组的toString就被重新定义 
```js
var a = [1,2,3] ; a.toString();
```

对于对象转换为数字，实际上是先将对象进行toString调用，再将字符串转换为数字。

## 内部属性[[Class]]

通过调用Object.prototype.toString()可以返回数据属性的class值

```js
Object.prototype.toString([1,2.3]) // "[object Array]"
```
可以看到返回一个字符串值，也就从侧面验证，所有的对象都是拥有toString方面，只是不同的类型构造函数，都重写了toString方法

