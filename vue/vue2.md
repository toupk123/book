# vue2.0

关于vue的一些问题和用法会记录在这里，不会完全的记录vue的api

## 关于vue的双向绑定

js提供了`Object.defineProperty` 方法，是对于object对象上属性进行的监听。

vue在处理data对象是，会在get中收集依赖，在set中派发依赖进行更新。

 

