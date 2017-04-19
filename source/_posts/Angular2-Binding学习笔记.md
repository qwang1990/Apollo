---
title: Angular2-Binding学习笔记
date: 2017-04-17 23:14:15
categories: 
- 前端
tags: 
- angular
---
# 纵观全局
绑定的类型可以根据数据流的方向分成三类：
    
     1.从数据源到视图
     2.从视图到数据源
     3.双向的从视图到数据源再到视图
     
## 下面是三种绑定的比较，里面具体名词不清楚没关系，后面会讲到      
![binding-categories](https://cloud.githubusercontent.com/assets/13915081/25138269/acb58114-248c-11e7-87ca-99e15450fe9f.png)
 

# 第一类绑定： 指令 ===> 视图目标
该绑定数据从指令流向模板中，主要有一下几种

    1.插值表达式
    2.绑定到Property上
    3.绑定到Attribute上
    4.绑定到class上
    5.绑定到style上
    
>注：在分别讲这几种类型之前先明确一个概念: Property vs Attribute (转自angular官网)
>HTML attribute 与 DOM property 的对比
>要想理解 Angular 绑定如何工作，重点是搞清 HTML attribute 和 DOM property 之间的区别。
>&emsp;&emsp;1.attribute 是由 HTML 定义的。property 是由 DOM (Document Object Model) 定义的。
>&emsp;&emsp;2.少量 HTML attribute 和 property 之间有着 1:1 的映射，如id。
>&emsp;&emsp;3.有些 HTML attribute 没有对应的 property，如colspan。
>&emsp;&emsp;4.有些 DOM property 没有对应的 attribute，如textContent。
>&emsp;&emsp;5.大量 HTML attribute看起来映射到了property…… 但却不像我们想的那样！
>最后一类尤其让人困惑…… 除非我们能理解这个普遍原则：
>attribute 初始化 DOM property，然后它们的任务就完成了。property 的值可以改变；attribute 的值不能改变。
>例如，当浏览器渲染&lt;input type="text" value="Bob"&gt;时，它将创建相应 DOM 节点， 其value property 被初始化为 “Bob”。
>当用户在输入框中输入 “Sally” 时，DOM 元素的value property 变成了 “Sally”。 但是这个 HTML value attribute 保持不变。如果我们读取 input 元素的 attribute，就会发现确实没变： input.getAttribute('value') // 返回 "Bob"。
>HTML attribute value指定了初始值；DOM value property 是当前值。
>disabled attribute 是另一个古怪的例子。按钮的disabled property 是false，因为默认情况下按钮是可用的。 当我们添加disabled attribute 时，只要它出现了按钮的disabled property 就初始化为true，于是按钮就被禁用了。
>添加或删除disabled attribute会禁用或启用这个按钮。但 attribute 的值无关紧要，这就是我们为什么没法通过 &lt;button disabled="false"&gt;仍被禁用&lt;/button&gt;这种写法来启用按钮。
>设置按钮的disabled property（如，通过 Angular 绑定）可以禁用或启用这个按钮。 这就是 property 的价值。
>就算名字相同，HTML attribute 和 DOM property 也不是同一样东西。

_简单来说就是HTML attribute是html元素定义的时候，定义完以后angular中操作改变的都是DOM property的值，所以除了插值表达式之外的绑定类型，在等号左边是目标名， 无论是包在括号中 ([]、()) 还是用前缀形式 (bind-、on-、bindon-)，这个目标名就是属性（Property）的名字。_

## 插值表达式
插值表达式是绑定中最简单的一种，就是把\{ \{ \} \}中的计算结果插入到HTML标签内的文本上，或给标签的属性赋值。
&lt;h3&gt;
  \{ \{ title \} \}
  &lt;img src=&quot;\{ \{heroImageUrl\} \}&quot; style=&quot;height:30px&quot;&gt;
  &lt;p&gt;The sum of 1 + 1 is \{ \{1 + 1\} \}&lt;/p&gt;
&lt;/h3&gt;

第一处是直接插入到HTML标签内的文本，第二处是给标签的属性赋值，第三处是也是给HTML标签内的文本插值，但是值是经过计算得来的

>#未完待续