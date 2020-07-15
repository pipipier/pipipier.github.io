---
title: BFC
date: 2017-6-03 14:23:54
categories:
 javaScript
---


1. DOM事件级别

    DOM0 element.onclick = function(){}

    DOM2 element.addEventListener('click', function(){}, false)

    DOM3 element.addEventListener('keyup', function()(), false)

    http://www.imooc.com/article/266179

2. DOM事件模型 捕获/冒泡

DOM事件模型分为捕获和冒泡。一个事件发生后，会在子元素和父元素之间传播（propagation）。这种传播分成三个阶段。

（1）捕获阶段：事件从window对象自上而下向目标节点传播的阶段；

（2）目标阶段：真正的目标节点正在处理事件的阶段；

（3）冒泡阶段：事件从目标节点自下而上向window对象传播的阶段。


3. DOM事件流

    事件通过捕获到达目标元素，从目标元素上传到window对象（冒泡过程）

4. 描述DOM事件捕获的具体流程

    window -> document -> html -> body -> ... -> 目标元素

    (冒泡流程反过来)

5. event对象的常见应用

    event.preventDefault() 阻止默认应用
    event.stopPropagation() 阻止冒泡
    event.stopImmediatePropagation()  既能阻止事件向父元素冒泡，也能阻止元素同事件类型的其它监听器被触发
    event.currentTarget 是事件绑定的元素,监听事件者
    event.target 引起触发事件的元素,事件的真正发出者

6. 自定义事件

    new Event / new CustomEvent
