---
title: Vue中实现拖放
date: 2017-06-13 21:52:11
tags:
- 拖放
- HTML5
- Vuejs
categories: 
- js
- HTML5
- Vuejs
---
最近在学习Vue，边做个小demo边学习。其中有一个小功能需要使用到拖放，顺便还学一下拖放。拖放是HTML5的标准，对着教程在普通的页面上很容易就实现了，但是vue中基本都是数据驱动，不推荐直接操作DOM。
<!-- more -->
## HTML5拖放
### 可拖动
首先，默认情况下，图像、链接和文本是可拖动的。而想让其他元素变为可拖动，需要设置`draggable`属性为`true`。
````html
<div draggable="true"></div>
````
### 拖放事件
拖放事件有的是在被拖动元素上触发的，而有的则是在放置目标上触发。当拖动某个元素时，将依次触发以下事件：
1. **dragstart**： 被拖动元素开始被拖动是触发
2. **drag**：      被拖动元素被拖动期间持续触发，类似于`mousemove`
3. **dragend**:    停止拖动元素时触发，无论被拖动元素是否放置在有效的目标上

当某个元素被拖动放在一个目标元素上时，放置目标则依次放生以下事件：
1. **dragenter**: 当拖动某个元素进入放置元素时触发
2. **dragover**： `dragenter`触发后，会触发`dragover`事件，并且如果拖动元素继续在放置目标范围内移动，该事件会持续触发
3. **drop或dragleave**： 在`dragover`事件后，如果是松开鼠标，被拖动元素放到放置目标上，触发`drop`事件；如果是继续拖动被拖动元素继续移动并离开了放置元素，这个放置目标元素就触发`dragleave`事件；

> 拖动元素经过各个元素时，这些元素虽然支持放置元素的事件，但是默认是不允许放置的，因此也就不会触发`drop`事件。需要在允许放置的元素中重写`dragenter`和`dragover`事件的默认行为，即使用`Event.preventDefault()`方法。

> 有些浏览器`drop`事件默认行为打开放到放置目标的URL。如拖放一个图片，就直接转到图片文件。因此也需要取消`drop`事件的默认行为。

### dataTransfer对象

在整个拖放过程中，需要进行数据交换的话，可以使用`dataTransfer`对象。`dataTransfer`对象作为event对象的属性，拥有两个主要方法`setData()`和`getData()`分别设置数据和获取数据。语法
````javascript
void dataTransfer.setData(format, data);
void dataTransfer.getData(format);
````

format是字符串类型，表示数据类型的；data也是字符串类型，即要保存的值。
> `dataTransfer`对象的两个属性`dropEffect`和`effectAllowed`，分别设置被拖动元素的放置效果和指定当元素被拖放时允许的效果。
> `dataTransfer`对象还有个`files`属性包含拖动到浏览器窗口的文件列表。可以用这个借口实现文件拖动上传。

简单的拖放例子可参考w3school的[HTML5拖放](http://www.w3school.com.cn/html5/html_5_draganddrop.asp "HTML5拖放")

<iframe width="100%" height="300" src="//jsrun.net/mNYKp/embedded/all/dark/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


## Vue拖放排序
小项目tomatodos中，todo列表我想加上一个拖放排序功能。就是todo列表中，允许用户直接拖动某一项，插入到另一项后。Vue中都是数据驱动的，因此在拖放后应该改变的是数据顺序，而不是直接操作dom。因此很自然想到的还是在`drop`后改变数据的顺序，拖动的元素则可以在`dragstart`中获取。我们的数据是

````javascript
lists: ['1: apple', '2: banana', '3: orange', '4: melon']
````
在html使用v-for渲染，使用`transition-group`组件加入动画效果
````html
<transition-group id='app' name="drog" tag="ul">
    <li draggable="true" v-for="(item, index) in lists" 
        @dragstart="dragStart($event, index)" @dragover="allowDrop" @drop="drop($event, index)" 
        v-bind:key="item">
        {{item}}
    </li>
</transition-group> 
````
> 每个li需要给定一个唯一的key，这样才能很好的使用过渡效果

索引在`dragstart`事件中传入，可以使用`dataTransfer`保存

````javascript
//开始拖动
dragStart(e, index){
    e.dataTransfer.setData('Text', index);
}
````
放置元素触发`drop`事件，在`drop`事件中，我们同时拥有放置目标元素的索引`index`，以及被拖动元素的索引`dragIndex`。比较简单的做法是使用一个新数组记录整个过程，最后把新数组赋给原数据变量。使用splice删除和插入，效果就是从前面拖到后面是插入放置元素后面，而从后面往前拖放是插入到放置元素前面。这样不需要再判断方向，也能得到比较好的效果。
````javascript
//放置
drop(e, index){
    //取消默认行为
    this.allowDrop(e);
    //使用一个新数组重新排序后赋给原变量
    let arr = this.lists.concat([]),
        dragIndex = e.dataTransfer.getData('Text');
        temp = arr.splice(dragIndex, 1);
    
    arr.splice(index, 0, temp[0]);

    this.lists = arr;

}

````

完整[demo](//jsrun.net/33YKp/embedded/all/dark/ "demo")

<iframe width="100%" height="300" src="//jsrun.net/33YKp/embedded/all/dark/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>