---
title: 对MVC、MVP、MVVM的理解
date: 2020-03-08 16:22:33
categories: [Framework]
tags:
---

## MVC

MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

<!-- more -->

![MVC](MVC.png)

### 数据关系

- View 接受用户交互请求
- View 将请求转交给Controller
- Controller 操作Model进行数据更新
- 数据更新之后，Model通知View更新数据变化
- View 更新变化数据

### 方式

所有方式都是单向通信

### 结构实现

View：使用 Composite模式
View和Controller：使用 Strategy模式
Model和 View：使用 Observer模式同步信息

### 使用

MVC中的View是可以直接访问Model的！从而，View里会包含Model信息，不可避免的还要包括一些业务逻辑。在MVC模型里，更关注的Model的不变，而同时有多个对Model的不同显示，及View。所以，在MVC模型里，Model不依赖于View，但是 View是依赖于Model的。不仅如此，因为有一些业务逻辑在View里实现了，导致要更改View也是比较困难的，至少那些业务逻辑是无法重用的。

## MVP

mvp的全称为Model-View-Presenter，Model提供数据，View负责显示，Controller/Presenter负责逻辑的处理。MVP与MVC有着一个重大的区别：在MVP中View并不直接使用Model，它们之间的通信是通过Presenter (MVC中的Controller)来进行的，所有的交互都发生在Presenter内部，而在MVC中View会直接从Model中读取数据而不是通过 Controller。

![MVP](MVP.png)

### 数据关系 - MVP

- View 接收用户交互请求
- View 将请求转交给 Presenter
- Presenter 操作Model进行数据更新
- Model 通知Presenter数据发生变化
- Presenter 更新View数据

### MVP的优势 - MVP

    1. Model与View完全分离，修改互不影响
    2. 更高效地使用，因为所有的逻辑交互都发生在一个地方—Presenter内部
    3. 一个Presenter可用于多个View，而不需要改变Presenter的逻辑（因为View的变化总是比Model的变化频繁）。

### 方式 - MVP

各部分之间都是双向通信

### 结构实现 - MVP

View：使用Composite模式
View和Presenter：使用Mediator模式
Model和Presenter：使用Command模式同步信息

### MVC和MVP区别

MVP与MVC最大的一个区别就是：Model与View层之间倒底该不该通信（甚至双向通信）

### MVC和MVP关系

项目开发中，UI是容易变化的，且是多样的，一样的数据会有N种显示方式；业务逻辑也是比较容易变化的。为了使得应用具有较大的弹性，我们期望将UI、逻辑（UI的逻辑和业务逻辑）和数据隔离开来，而MVP是一个很好的选择。
Presenter代替了Controller，它比Controller担当更多的任务，也更加复杂。Presenter处理事件，执行相应的逻辑，这些逻辑映射到Model操作Model。那些处理UI如何工作的代码基本上都位于Presenter。
MVC中的Model和View使用Observer模式进行沟通；MVP中的Presenter和View则使用Mediator模式进行通信；Presenter操作Model则使用Command模式来进行。基本设计和MVC相同：Model存储数据，View对Model的表现，Presenter协调两者之间的通信。在 MVP 中 View 接收到事件，然后会将它们传递到 Presenter, 如何具体处理这些事件，将由Presenter来完成。
如果要实现的UI比较复杂，而且相关的显示逻辑还跟Model有关系，就可以在View和 Presenter之间放置一个Adapter。由这个 Adapter来访问Model和View，避免两者之间的关联。而同时，因为Adapter实现了View的接口，从而可以保证与Presenter之 间接口的不变。这样就可以保证View和Presenter之间接口的简洁，又不失去UI的灵活性。

### 使用 - MVP

MVP的实现会根据View的实现而有一些不同，一部分倾向于在View中放置简单的逻辑，在Presenter放置复杂的逻辑；另一部分倾向于在presenter中放置全部的逻辑。这两种分别被称为：Passive View和Superivising Controller。

## MVVM

MVVM是Model-View-ViewModel的简写。微软的WPF带来了新的技术体验，如Silverlight、音频、视频、3D、动画……，这导致了软件UI层更加细节化、可定制化。同时，在技术层面，WPF也带来了 诸如Binding、Dependency Property、Routed Events、Command、DataTemplate、ControlTemplate等新特性。MVVM（Model-View-ViewModel）框架的由来便是MVP（Model-View-Presenter）模式与WPF结合的应用方式时发展演变过来的一种新型架构框架。它立足于原有MVP框架并且把WPF的新特性糅合进去，以应对客户日益复杂的需求变化。

![MVVM](MVVM.png)

### 数据关系 - MVVM

- View 接收用户交互请求
- View 将请求转交给ViewModel
- ViewModel 操作Model数据更新
- Model 更新完数据，通知ViewModel数据发生变化
- ViewModel 更新View数据

### 方式 -MVVM

双向绑定。View/Model的变动，自动反映在 ViewModel，反之亦然。

### 使用 - MVVM

- 可以兼容你当下使用的 MVC/MVP 框架。
- 增加你的应用的可测试性。
- 配合一个绑定机制效果最好。

### MVVM优点

MVVM模式和MVC模式一样，主要目的是分离视图（View）和模型（Model），有几大优点:

1. 低耦合。View可以独立于Model变化和修改，一个ViewModel可以绑定到不同的”View”上，当View变化的时候Model可以不变，当Model变化的时候View也可以不变。
2. 可重用性。你可以把一些视图逻辑放在一个ViewModel里面，让很多view重用这段视图逻辑。 
3. 独立开发。开发人员可以专注于业务逻辑和数据的开发（ViewModel），设计人员可以专注于页面设计，生成xml代码。 
4. 可测试。界面素来是比较难于测试的，而现在测试可以针对ViewModel来写。

## mvc,mvp,mvvm三者演化

MVC => MVP => MVVM

参考：

http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html
