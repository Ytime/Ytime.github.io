---
title: Vue原理及实现
date: 2017-08-02 22:53:22
tags:
- Vuejs
categories: 
- js
- Vuejs
---
学了一小段时间的Vue，之前多多少少有了解一些它的原理，但是一直没能有机会梳理一遍。而且之前在面试中也被问到过，才发现自己居然说不出来。因此这样让我下定决心自己按照Vue的原理实现一遍简单的MVVM框架。
<!-- more -->
本文实现除了参考[官方源码Vuejs](https://github.com/vuejs/vue "Vuejs")，还参考[开源仓库](https://github.com/DMQ/mvvm)，在此做出说明。
## 响应式原理
> 把一个普通JavaScript对象传给Vue实例的data选项，Vue将遍历此对象所有的属性，并使用Object.defineProperty把这些属性全部转为getter/setter。Object.defineProperty 是仅ES5支持，且无法shim的特性，这也就是为什么Vue不支持IE8以及更低版本浏览器的原因

[了解更多defineProperty](https://segmentfault.com/a/1190000007434923)

Vue.js是采用`Object.defineProperty`的`getter`和`setter`，也叫数据劫持，并结合观察者模式来实现数据绑定的。
![image](https://sfault-image.b0.upaiyun.com/349/066/349066591-59157f40dbd7a_articlex)
* Observer数据监听器，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者，内部采用Object.defineProperty的getter和setter来实现。
* Compile指令解析器，它的作用对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数。
* Watcher订阅者， 作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数。
* Dep消息订阅器，内部维护了一个数组，用来收集订阅者（Watcher），数据变动触发notify函数，再调用订阅者的update方法。

## 实现
### Observer
首先我们使用ES5的`Object.defineProperty`来实现属性的监听。对要绑定的`data`对象的属性依次设置`getter`和`setter`。`data` 子属性也需要监听。

````javascript
var Observer = function Observer(data){
    this.data = data;
    //遍历属性设置响应
    this.walk(data);
}

Observer.prototype = {
    //遍历属性设置响应
    walk: function(obj){
        Object.keys(obj).forEach(key => this.defineReactive(obj, key, obj[key]));
    },
    //设置属性响应
    defineReactive: function(data, key, val){
        // 递归监听子属性
        observe(val);
        // 定义setter和getter
        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get: function () {
                return val;
            },
            set: function(newVal){
                if (val === newVal) return;
                console.log(key + ': ' + val + ' --> ' + newVal);
                if (typeof newVal == 'object') {
                    observe(newVal);
                }
                val = newVal;
            }
        })
    }
}
````
我们监听如下`data`对象，一切都很好，但是使用Array的push等方法并不会被监听到。

````javascript
let data = {
    name: 'hsj',
    msg: 'hello',
    todos: ['']
};
observe(data);
data.msg = 'hi';                 //msg: hello --> hi
data.todos = ['study', 'work'];  //todos: --> study,work
data.todos[0] = 'sleep';         //0: study --> sleep
data.todos.push('sleep');        //3 没有监听到变化
````
为了实现监听数组，我们自然要改写操作数组的方法，但是又不能改写原生Array原型中的方法。简单的方法是使用`Object.create`继承Array的原型创建一个新对象`arrFakeProto`，用这个对象替换需要监听的数组的原型。
````javascript
const arrProto = Array.prototype,
    arrFakeProto = Object.create(arrProto),
    arrMethods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'];
arrMethods.forEach(function (method){
    let original = arrProto[method];
    arrFakeProto[method] = function(){
        let arr = this.concat(),
            //这里我们就能监听到变化了，调用原生的数组方法
            res = original.apply(this, arguments);
        console.log(arr + ' --> ' + this);
        return res;
    }
})
````
看看操作数组的监听效果
````javascript
let arr = ['study', 'work'];
Object.setPrototypeOf(arr, arrFakeProto);
arr.push('sleep');     //study,work --> study,work,sleep   3
Array.isArray(arr)     //true
````

综上，我们得到完整的observe代码
````javascript
var arrProto = Array.prototype,
    arrFakeProto = Object.create(arrProto),
    arrMethods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'];
arrMethods.forEach(function (method){
    var original = arrProto[method];
    arrFakeProto[method] = function(){
        var arr = this.concat(),
            //这里我们就能监听到变化了，调用原生的数组方法
            res = original.apply(this, arguments);
        console.log(arr + ' --> ' + this);
        return res;
    }
})


var Observer = function Observer(data){
    this.data = data;
    // 遍历属性设置响应
    if (Array.isArray(data)) {
        // 替换数组原型
        Object.setPrototypeOf(data, arrFakeProto);
        this.observeArray(data);
    }
    this.walk(data);
}

Observer.prototype = {
    //遍历属性设置响应
    walk: function(obj){
        Object.keys(obj).forEach(key => this.defineReactive(obj, key, obj[key]));
    },
    //设置属性响应
    defineReactive: function(data, key, val){
        // 递归监听子属性
        observe(val);
        // 定义setter和getter
        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get: function () {
                return val;
            },
            set: function(newVal){
                if (val === newVal) return;
                console.log(key + ': ' + val + ' --> ' + newVal);
                if (typeof newVal == 'object') {
                    observe(newVal);
                }
                val = newVal;
            }
        })
    },
    //监听数组
    observeArray: function(items){
        items.forEach(item => observe(item));
    }
}

//监听data
function observe(data){
    if (typeof data !== 'object') {
        return;
    }
    return new Observer(data);
}
````
以上代码中，array和object分开监听，array监听数组操作方法，但是还有个问题，那就是新增加的数组元素，没有被监听。当然其实在重写push等方法中也可以重新对添加的元素进行监听。
````javascript
let data = {
    name: 'hsj',
    msg: 'hello',
    todos: ['']
};
observe(data);
data.msg = 'hi';                 //msg: hello --> hi
data.todos = ['study', 'work'];  //todos: --> study,work
data.todos[0] = 'sleep';         //0: study --> sleep
data.todos.push('sleep');        //sleep,work --> sleep,work,sleep  3
````

至此，简单的监听已经实现了，但是还需要做一件事情，那就是在监听到变化后通知订阅者`Watcher`，因此还需要实现上文说的消息订阅器Dep，内部维护一个数组，用来收集订阅者（Watcher），数据变动触发notify函数，再调用订阅者的update方法。

