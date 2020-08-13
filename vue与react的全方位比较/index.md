# vue2与React源码浅析

从使用两个框架第一步开始，也就是初始化挂在到页面的地方。

```js
// vue 
new Vue({
  el:"#app",
  // 一些其他的配置
})

// react
import {render} from "react-dom"
render(<App />,ELEMENT_NODE)

```

从这里可以看出两个框架的不同，对于vue来说，这里不仅是在进行挂载，同时进行了一个vue实例化，一些全局属性的声明操作就是在这里进行的。

react则是不同，并没有对外暴露过多东西，这是将一个组件挂在到了节点上，关于react实例化这里完全对外隐藏。

进入一步查看两个框架的初始化操作

```js
// vue 源码地址 src-core-instace-index.js
function Vue (options) {
  this._init(options)
}
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

// 核心代码显然就是这里的this._init操作，vue实例化的操作，可以看到vue是通过mini形式，直接挂在到vue的原型对象上
function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    //...
  }
  // ...
}

// react
function render(element,container,callback){
	  return legacyRenderSubtreeIntoContainer(
    null, // parentComponent
    element,
    container,
    false, //是否ssr服务端渲染
    callback,
  );
  //直接调用了legacyRenderSubtreeIntoContainer方法
}

```

下面将详细讲解两个框架的init过程。

```js
//首先从react开始
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  container: Container,
  forceHydrate: boolean,
  callback: ?Function,
) {
  let root: RootType = (container._reactRootContainer: any);
   // 初始化自然没有rootrContainer
  let fiberRoot;
  if (!root) {
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    fiberRoot = root._internalRoot;
    // 可以看到在初始化的时候，直接获取到fiberRoot
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
		//...
  }
  return getPublicRootInstance(fiberRoot);
}

// legacyCreateRootFromDOMContainer -> createLegacyRoot->createRootImpl->
// ...->createFiberRoot
function createLegacyRoot(){
    return new ReactDOMBlockingRoot(container, LegacyRoot, options);
} //到这一步就可以看到，react的模式为block模式
	// 在ReactDOMBlockingRoot 中定义了_internalRoot属性

// 直到 createFiberRoot才看到fiber真正的数据结构
function createFiberRoot(
  containerInfo: any, // 挂在到节点的dom信息
  tag: RootTag,	//可以看作react的模式 BlockingRoot = 1
  hydrate: boolean,
  hydrationCallbacks: null | SuspenseHydrationCallbacks,){

}


```



