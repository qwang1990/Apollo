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

## 绑定到Property 
绑定到property分为两类:
- 元素的property
    &lt;button \[disabled\]=&quot;isUnchanged&quot;&gt;Cancel is disabled&lt;/button&gt;
- 指令(组件)的property
     + 设置angular自带指令的属性(这个在属性型指令和结构型指令中会细讲)   
        &lt;div \[ngClass\]=&quot;classes&quot;&gt;\[ngClass\] binding to the classes property&lt;/div&gt;
     + 设置自定义的组件(指令)的属性
         &lt;hero-detail \[heroName\]=&quot;hero&quot;  &gt;&lt;/hero-detail&gt; 
         此时hero-detail的代码如下:
         ```angular2html
           @Component({
             selector: 'hero-detail',
             template: `
             <h1>heroName: {{heroName}}</h1>
           `,
           })
           export class HeroDetailComponent  {
             @Input() heroName:string;
           
           }
          ```
          即\[heroName\]就是组件中的待输入的property，类似java中的一个类的构造参数
> 绑定到property的过程是单向的，即从组件的数据流向目标属性的数据，如果必须读取目标元素上的属性或调用它的某个方法，
> 得用另一种技术 --- ViewChild 和 ContentChild。

## 绑定到Attribute
&emsp;&emsp;绑定到Attribute的原因大多是元素没有属性可绑的时候，所以必须使用attribute绑定。table中的 colspan/rowspan 等 attribute。 它们是纯粹的 attribute，没有对应的属性可供绑定。

```angular2html
    <table border=1>
      <!--  expression calculates colspan=2 -->
      <tr><td [attr.colspan]="1 + 1">One-Two</td></tr>
      <!-- ERROR: There is no `colspan` property to set!
        <tr><td colspan="{{1 + 1}}">Three-Four</td></tr>
      -->
      <tr><td>Five</td><td>Six</td></tr>
    </table> 
```

## 绑定到CSS上
借助 CSS 类绑定，可以从元素的class attribute 上添加和移除 CSS 类名。
```angular2html
<!-- toggle the "special" class on/off with a property -->
<div [class.special]="isSpecial">The class binding is special</div>
```
## 绑定到style上
样式绑定的语法与属性绑定类似
```angular2html
    <button [style.color]="isSpecial ? 'red': 'green'">Red</button>
    <button [style.background-color]="canSave ? 'cyan': 'grey'" >Save</button>
```

# 第二类绑定：视图目标 ===> 指令
## 基本语法 :圆括号中的名称 —— 比如(click) —— 标记出目标事件。在下面例子中，目标是按钮的 click 事件。 
```angular2html
    <button (click)="onSave($event)">Save</button>
```
绑定会通过名叫$event的事件对象传递关于此事件的信息(包括数据值)

## 使用EventEmitter实现自定义事件
通常，指令使用 Angular EventEmitter 来触发自定义事件。 指令创建一个EventEmitter实例，并且把它作为属性暴露出来。 指令调用EventEmitter.emit(payload)来触发事件，可以传入任何东西作为消息载荷。 父指令通过绑定到这个属性来监听事件，并通过$event对象来访问载荷。

假设HeroDetailComponent用于显示英雄的信息，并响应用户的动作。 虽然HeroDetailComponent包含删除按钮，但它自己并不知道该如何删除这个英雄。 最好的做法是触发事件来报告“删除用户”的请求。、

```angular2html
    template: `
    <div>
      <img src="{{heroImageUrl}}">
      <span [style.text-decoration]="lineThrough">
        {{prefix}} {{hero?.name}}
      </span>
      <button (click)="delete()">Delete</button>
    </div>`
    
    // This component make a request but it can't actually delete a hero.
    deleteRequest = new EventEmitter<Hero>();
    
    delete() {
      this.deleteRequest.emit(this.hero);
    }
```
组件定义了deleteRequest属性，它是EventEmitter实例。 当用户点击删除时，组件会调用delete()方法，让EventEmitter发出一个Hero对象。

现在，假设有个宿主的父组件，它绑定了HeroDetailComponent的deleteRequest事件。

```angular2html
<hero-detail (deleteRequest)="deleteHero($event)" [hero]="currentHero"></hero-detail>
```
当deleteRequest事件触发时，Angular 调用父组件的deleteHero方法， 在$event变量中传入要删除的英雄（来自HeroDetail）。
> 上面这个例子就可以看成hero-detail组件自定义了一个deleteRequest事件（可以类比button元素的click事件）

# 第三类绑定：视图目标 <===> 指令 (双向邦定)
简单来说双向绑定就是既要设置 _元素属性_ ，又要监听元素事件变化。
Angular 为此提供一种特殊的双向数据绑定语法：\[(x)\]。 \[(x)\]语法结合了属性绑定的方括号\[x\]和事件绑定的圆括号(x)
当一个元素拥有可以设置的属性x和对应的事件xChange时，解释\[(x)\]语法就容易多了。 下面的SizerComponent符合这个模式。它有size属性和伴随的sizeChange事件：

_sizer.component.ts_
```angular2html
import { Component, EventEmitter, Input, Output } from '@angular/core';
@Component({
  selector: 'my-sizer',
  template: `
  <div>
    <button (click)="dec()" title="smaller">-</button>
    <button (click)="inc()" title="bigger">+</button>
    <label [style.font-size.px]="size">FontSize: {{size}}px</label>
  </div>`
})
export class SizerComponent {
  @Input()  size: number | string;
  @Output() sizeChange = new EventEmitter<number>();
  dec() { this.resize(-1); }
  inc() { this.resize(+1); }
  resize(delta: number) {
    this.size = Math.min(40, Math.max(8, +this.size + delta));
    this.sizeChange.emit(this.size);
  }
}
```
这段代码重在于:
- size是一个输入属性用于初始
- 点击smaller或bigger会触发size大小的变化
- size大小变化后会触发sizeChange事件(这里用到了上文中说到的EventEmitter)

_app.component.ts_
```angular2html
<my-sizer [(size)]="fontSizePx"></my-sizer>
<div [style.font-size.px]="fontSizePx">Resizable Text</div>
```
在这列AppComponent的fontSizePx就被双向绑定到了SizerComponent上
- SizerComponent的size的初始值就是AppComponent的fontSizePx
- 在SizerComponent组件内通过点击按钮更新size的大小会影响fontSizePx的大小，最终会改变Resizable Text的显示

双向绑定语法实际上是属性绑定和事件绑定的语法糖。 Angular将SizerComponent的绑定分解成这样：
```angular2html
<my-sizer [size]="fontSizePx" (sizeChange)="fontSizePx=$event"></my-sizer>
```

我们希望能在像&lt;input&gt;和&lt;select&gt;这样的 HTML 元素上使用双向数据绑定。 可惜，原生 HTML 元素不遵循x值和xChange事件的模式。

幸运的是，Angular 以 NgModel 指令为桥梁，允许在表单元素上使用双向数据绑定。

>在使用ngModel指令之前需要先引入FormsModule

现在假设我们需要把一个input的value属性和输入变更都绑定到组件的myName属性。 使用以下方式也可以满足这个需求:
```angular2html
<input [value]="myName"
       (input)="myName" >
```
但是这个很不好用，因为你需要记住每一个标签的设置值得属性和变更时需要绑定的事件
使用ngModel你可以用下面的方式完成同样的功能,它使用ngModel这个指令屏蔽了不同标签的设置属性和变更事件。
```angular2html
<input
  [ngModel]="myName"
  (ngModelChange)="myName">
```
上面代码可以简写为:
```angular2html
<input [(ngModel)]="myName">
```
使用第二种方法比较简单，但是它的局限是只能完成属性值的绑定，如果想要实现更复杂的功能还是需要使用第一种写法。

因为HTML的元素实现细节不同，所以angular2只对基本的HTML Form元素提供了支持。

>源码https://github.com/qwang1990/angular2/tree/master/angular_binding







