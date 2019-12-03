# 深拷贝和浅拷贝  

## 什么是深拷贝和浅拷贝？  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;js中变量储存着值，我们将一个变量的值复制给另一个变量时，就会发生值的拷贝，但因为js在储存值分为两种方式。  

        1. 基础类型，储存在执行栈中，执行函数退出后，变量将会清除  
        2. 引用类型，储存在堆内存中，不会随着函数退出立刻清除。

 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于这样的区别，所以js在进行拷贝时，将会出现明显的区别，基础类型的赋值，是将值进行拷贝，但是存在堆内的引用类型，则只是将一个引用地址值，拷贝给了变量。

        这种只拷贝了引用地址的拷贝方式，就是浅拷贝。所以深拷贝的意思，就是完完全全将引用内存的值，再拷贝一份，在堆内存中分配一个空间储存。

## 实现深拷贝

实际开发中，应用最多肯定是深拷贝。

###  JSON.parse(JSON.stringify(obj))

这种是最简单的深拷贝，几乎可以说应用到所有的场景，但是同时也会有很多的缺陷。

// 当然，这里其实也涉及到了可枚举和不可枚举属性，文章下面将会展开。这种方式是无法拷贝原型上面上的属性。

1. 会忽略undefined(因为undefinde不是json通用值)

2. 会忽略symbol(同理)

3. 不能序列化函数

4. 不能解决循环引用的对象

5. 不能正确处理new Date()

6. 不能正确处理正则


### 基本版本

```js

function clone(obj){
       let cloneObj = {};
       for(const key in obj){
              cloneObj[key] = obj[x]
       }
       return cloneObj
}

// 这种方式没有考虑深层次结构的对象，使用递归
function clone(obj){
       if(typeof obj === 'object'){
       let cloneObj = Array.isArray(obj)?[]:{};//考虑数组
       for(const key in obj){
              cloneObj[key] = clone(obj[x])
       }
       return cloneObj
       } else {
       return obj
       }
}

```


### 循环引用
```js
const target = {
    field1: 1,
    field2: undefined,
    field3: {
        child: 'child'
    },
    field4: [2, 4, 8]
};
target.target = target;

```

为了解决循环引用的问题，我们可以使用Map结构

1. 检查map中有无克隆过的对象

2. 有 直接返回

3. 没有 将当前对象作为key，克隆对象作为value

4. 继续克隆

```js
function clone(target, map = new Map()) {
    if (typeof target === 'object') {
        let cloneTarget = Array.isArray(target) ? [] : {};
        if (map.get(target)) {
            return map.get(target);
        }
        map.set(target, cloneTarget);
        for (const key in target) {
            cloneTarget[key] = clone(target[key], map);
        }
        return cloneTarget;
    } else {
        return target;
    }
};
// 这里也可以weakMap，因为weakMap是弱引用，可以直接在下一次垃圾回收掉。
```








