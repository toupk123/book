# 面试题

用来记载一些可以延伸的面试题，一些简单的面试题不会记录在内

## 元素在窗口中消失以及垂直水平居中方法，还有flex布局的理解

- 元素消失的方案先列出来， `display:none`和`visibility: hidden;`的区别，拓展到`vue`框架的`v-if`和`v-show`的区别，可以搭配回流和重绘来讲解

### 关于vue中的v-if和v-show

其实也就是display:none和visibility: hidden，v-if会减少初始化dom元素加载，但是切换时会导致回流，visibility:hidden不会导致回流，但是初始化加载dom，导致初始化加载量变大。

### 关于回流和重绘

#### 回流必将引起重绘，重绘不一定会引起回流

因为按照现代浏览器流式布局，回流会导致render tree重新渲染，一定会进行重绘

当`Render Tree`中部分或全部元素的尺寸、结构、或某些属性发生改变时，浏览器重新渲染部分或全部文档的过程称为回流

下面内容会导致回流:

1. 页面首次渲染

2. 浏览器窗口大小发生改变

3. 元素尺寸或位置发生改变

4. 元素内容变化（文字数量或图片大小等等）

5. 元素字体大小变化

6. 添加或者删除**可见**的`DOM`元素

7. 激活`CSS`伪类（例如：`:hover`）

8. 查询某些属性或调用某些方法:

clientWidth、clientHeight、clientTop、clientLeft

offsetWidth、offsetHeight、offsetTop、offsetLeft

scrollWidth、scrollHeight、scrollTop、scrollLeft

scrollIntoView()、scrollIntoViewIfNeeded()

getComputedStyle()

getBoundingClientRect()

scrollTo()

