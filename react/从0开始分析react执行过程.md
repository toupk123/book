# 执行过程

从react执行写下一个组件开始

```js
class Test extends React.Component{
  render(){
    return <div>测试</div>
  }
}
```

首先这里得明白我们使用jsx语法时，其实实际上使用的`React.createElement`.

## Component

```js
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
} 
```

`Test`组件实例化之后，我们看看他的数据结构

```js
context: {}
props: {history: {…}, location: {…}, match: {…}, staticContext: undefined}
refs: {}
state: null
updater: {
  enqueueForceUpdate: ƒ (inst, callback),
	enqueueReplaceState: ƒ (inst, payload, callback),
	enqueueSetState: ƒ (inst, payload, callback),
	isMounted: ƒ isMounted(component)
}
```

这里的updater非常的重要，在后面讲进行讲解。

### createElement

```js
function createElement(type, config, children) {
  let propName;
  const props = {};
  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    props.children = childArray;
  }
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
} 
```

在经过createElement函数处理之后，可以得到一个对象:

```js
{
  $$typeof: Symbol(react.element)
  key: null
  props: {}
  ref: null
  type: ƒ Test()
  _owner: null
  _store: {validated: false}
}
// 注意这里并没有产生一个虚拟dom
```

## 从render开始