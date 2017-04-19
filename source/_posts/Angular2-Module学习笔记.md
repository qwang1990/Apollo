---
title: Angular2 Module学习笔记
date: 2017-04-08 21:36:19
categories: 
- 前端
tags: 
- angular
---
{% asset_img angular.jpg  angular %}

# 概念
Angular 模块能帮你把应用组织成多个内聚的功能块，可以类比java的package

# 定义
```angular2html
    @NgModule({
      imports:      [ CommonModule ],
      declarations: [ TitleComponent ],
      exports:      [ TitleComponent ],
      providers:    [ UserService ],
      <!--bootstrap:    [ AppComponent ]-->
    })
```
Angular 模块是一个由@NgModule装饰器提供元数据的类，元数据包括：
- imports: 导入其它模块，从其它模块中获得本模块所需的组件、指令和管道
- declarations: 声明哪些组件、指令、管道属于该模块
- exports: 公开某些类，以便其它的组件模板可以使用它们
- provider: 在应用程序级提供服务，以便应用中的任何组件都能使用它
- bootstrap: 把指定组件标记为引导 (bootstrap) 组件。当 Angular 引导应用时，它会在 DOM 中渲染AppComponent，并把结果放进index.html的<my-app>元素标记内部。这个元数据只有根模块有。




# 模块基本使用
- 声明组件和指令: 定义的组件和指令必须在模块中声明了，模板才能识别组件和指令对应的标签
```angular2html
  例如我们定义指令HighlightDirective：
    import { Directive, ElementRef } from '@angular/core';
 
    @Directive({ selector: '[highlight]' })
    /** Highlight the attached element in gold */
    export class HighlightDirective {
      constructor(el: ElementRef) {
        el.nativeElement.style.backgroundColor = 'gold';
        console.log(
          `* AppRoot highlight called for ${el.nativeElement.tagName}`);
      }
    }
    
    在使用 <h1 highlight>{{title}}</h1> 前必须要在我们的module的declaration中加入如下定义：
    declarations: [
      ... 
      HighlightDirective,
      ...
    ],
``` 

- 服务提供商
  模块可以往应用的“根依赖注入器”中添加提供商，让那些服务在应用中到处可用,这个和在组件中提供服务不同。<font color=#ff0000>组件中提供的服务是分层级的。</font>这个在多级注入的时候会讲到。


# 特性模块
- 为什么要有多个模块
&emsp;&emsp;随着应用越来越大，如果只是用一个根模块就会出现很多问题。 比如指令冲突，我们在一个类中定义了ADirective，然后再另一个类中定义了同名的ADirective，并且他们的selector一样都是'sameName'.当我们在别处使用这个&lt;sameName ... &gt;时就会出现，后一个declaration的指令会覆盖前一个declaration的指令。

- 什么是特性模块
特性模块和根模块一样都是是带有@NgModule装饰器及其元数据的类，它和根模块有着同一个依赖注入器。不同点是: 
        
       1.我们引导根模块来启动应用，但导入特性模块来扩展应用。
       2.特性模块可以对其它模块暴露或隐藏自己的实现。(通过exports来控制)

- 一个使用特性模块的小技巧---重新导出
场景如下:
&emsp;&emsp;  你有一个特性模块A, 它引入了CommonModule和FormsModule在自身模块内使用, 这时另一个模块B导入模块A, 而且B也需要使用CommonModule和FormsModule, 正常情况下B还需要导入CommonModule和FormsModule这两个模块, 
但如果模块A将CommonModule和FormsModule export出去, 这时B就只需导入A模块即可。

# 没图说个XX
![angular_module 001](https://cloud.githubusercontent.com/assets/13915081/25060334/0efbeb14-21cd-11e7-87af-9a08a372952d.png)

本例中RootModule分别引入了ModuleM和ModuleW
- ModuleM 声明了两个组件, 导入了FormModule, 并提供了ServiceMA服务,最后它导出了自己的其中一个组件ComponentMA和ForModule
- ModuleW 声明了两个组件, 导入了RouterModule, 并提供了ServiceWA服务,最后它只导出了自己的其中一个组件ComponentMA

此时会有以下结果:

    1. ModuleM 中可以使用自己声明的组件(即在自己的模板中使用组件的selector标签)和FormModule中的功能。
    2. ModuleW 中可以使用自己声明的组件(即在自己的模板中使用组件的selector标签)和RouterModule中的功能。
    3. 全局中都能使用 serviceA,ServiceMA,serviceWA。以为模块级别的依赖注入器是同一个。
    4. 在RootModule中可以使用FormModule,ComponentMA和ComponentWB。


源码：https://github.com/qwang1990/angular2/tree/master/angular_module









