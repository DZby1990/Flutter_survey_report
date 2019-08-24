#Flutter调研报告


##一、 Flutter简介

Flutter是google公司开发的一款基于Dart语言的全新一代跨平台，开源的UI框架。它最早是在2015年Dart开发者峰会上与大家见面，在此之后2017年5月发布了第一个Aplpha 版本，并于2018年12月初发布了1.0稳定版。
Google发布这个框架的初衷，就是结合自身多年来在各个方向，包含但不止于前端，移动端的经验，以及友商或失败或成功的经验，结合最新的一些框架理念，为下一代大前端统一UI框架发出的又一次冲击。为此铺路，Google也已经宣布将在Fuchsia 操作系统中扶正Flutter成为其官方开发框架。而Fuchsia 操作系统本身相比android，可以做到更轻量，对硬件和内存要求更低，本身定位是Google面对下一代物联网的主打操作系统。因此可见Google对其Flutter框架性能的信心。

##二、 前车之鉴

在此之前，市面上已经出现了以ReactNative，weex为代表的跨平台方案。但是就目前来说，这两套方案都远没有达到人们的预期，甚至weex已经走向了生命周期的尽头。那么Flutter和他的前者比到底有哪些不同呢？
从大了讲，RN和weex以及其他的几乎所有flutter之前出现的跨平台方案，都无外乎两种设计思路。
###1. H5时代，
这种类型的跨平台方案，其实就是将一个web网页尽可能的模仿移动端原生的页面效果，依托android和iOS各自平台的WebView网页容器控件，来打开网页，从而实现跨平台的效果。这种方案的缺点就很明显，它在性能上是远远不能和原生UI框架相提并论，而且观感效果差强人意，交互上也受很大限制。
###2. 以RN为代表的时代，
为了解决H5时代性能上的巨大差距，Facebook于15年开源了这套UI框架，包括weex在内，他们解决性能问题的法宝，按通俗了讲就是代码映射。以RN为例。
 
 ![image](https://github.com/DZby1990/Flutter_survey_report/tree/master/pic/1.png "")
RN本身使用起来，其实和普通的前端框架并无二致，前端开发者的入门成本很低，开发者可以大部分使用JS代码就构建起一个跨平台APP。RN 会把应用的JS代码（包括依赖的framework）编译成一个js文件（一般命名为index.android.bundle),  RN的整体框架目标就是为了解释运行这个js 脚本文件，如果是js 扩展的API， 则直接通过bridge调用native方法; 如果是UI界面， 则映射到virtual DOM这个虚拟的JS数据结构中，通过bridge 传递到native ， 然后根据数据属性设置各个对应的真实native的View。 bridge是一种JS 和 JAVA代码通信的机制， 用bridge函数传入对方module 和 method即可得到异步回调的结果。
用一句话概括，就是把前端代码通过virtual DOM生成android或者iOS的原生VIew控件。所有的RN代码，和原生代码，都是一一对应的关系。
听上去很美，RN对比H5确实大幅提高了应用的性能，但是经过了几年的发展，也逐渐走进了死胡同。
####第一，他的自由度不及原生应用。
由于它的实现方式是将RN代码变成原生代码来进行展示，那么必然存在原生可以做而RN无法覆盖到的功能，事实也确实如此，一些复杂的界面效果和动画效果，RN都是无法做到的，而且在功能上也必然滞后于原生平台。
####第二，还是性能问题。
android的碎片化，导致RN的一些效果或者在较多bridge调用的情况下，确实在中低端机型上表现不佳，毕竟RN归根结底还是需要一层转译，无可避免的会造成性能消耗
####第三，最大的问题。
当大家以为RN的引入会节约一半人员并欣然投入其中之后，却发现RN的使用缺导致了人员的上升。其主要原因，还是在与iOS平台与Android平台的差异性以及Android平台的碎片化。相当大部分大量使用RN进行开发的团队都表示，如果要与原生一样的标准完成需求，那就需要额外的适配工作，并且不可避免的会导致Android和iOS平台使用了不同的代码分支。而且RN底层库的更新，经常会带来两个平台不同的坑以及一些老功能代码的弃用。

##三、 Flutter的基本架构

  
 ![image](https://github.com/DZby1990/Flutter_survey_report/tree/master/pic/2.png "")
从官网拿来的Flutter框架图。其实我们可以大概了解到，framwork层当中有自己独立的动画，绘图，手势库来绘制界面及效果。Rendering是负责构建整个界面的UI控件树。而Widget则是一个面向前端开发者的核心概念，它是Flutter所有控件的父类， Material 和Cupertino则是在Widget上实现的两种风格的控件。
Engine层则是由C和C++代码构成。从名称上看，我们除了Dart引擎，Dart虚拟机引擎外，还能看到系统事件，文本，绘制以及平台桥接引擎等，这点也是Flutter进行的一个很大突破，在下文中会有详细的描述。

##四、 Flutter进行的突破
由于本人对iOS不甚了解，这部分将重点举例讲述Flutter作为Google亲儿子，在跨平台Android端所做的突破。当然，android和iOS的实现也是大致一致的。
首先，讲在第一句。Flutter的实现思路，和之前的两代跨平台方案是有根本不同的。
 ![image](https://github.com/DZby1990/Flutter_survey_report/tree/master/pic/3.png "")
 
通过上图，我们可以粗略的知道，Flutter其实是一个完全独立的UI框架，它本身的编译和UI绘制，完全不依赖原生的Android和iOS框架。
针对Android的实现，它本身在android端的入口为FlutterActivity。我们可以简单的认为，在原生的维度，一个纯Flutter应用，其实就是一个单一Activity界面的应用。
而它完成一整套UI框架，依赖的则是FlutterView这个View控件。这个FlutterView本身继承自android原生的SurfaceView。SurfaceView本身继承自View，它的特点就是每个SurfaceView都有单独的线程运行和单独绘制，不会占用主线程资源。它本身在原生中的作用是用于视频播放，承载游戏界面内容以及复杂动画效果。可以说天生就是拿来让大家折腾的。
对于Flutter维度，每一个FlutterView其实就是一个单独的界面，Flutter本身维护的界面栈，就是一个个的FlutterView。而Flutter正是在FlutterView中承载了一整套完全与原生无关的UI框架。iOS的实现也大致差不多。
那么问题来了，针对之前的缺点和不足，它到底带来了哪些变化？
###第一，性能提升。
由于Flutter完全与原生UI框架无关，所以不存在RN当中JS通过bridge与原生交互过多导致的性能问题。在网上的多方性能评测来看，Flutter与原生的性能基本上是不相上下的水平，甚至在一些低端机上，出现了Flutter性能优于原生的情况。如果怎么说你还不相信，那么 Google I/O 2019上，谷歌为大家带来了Flutter Developer Quest。这是一款完全由Flutter开发的，跨平台的，2d开源RPG类游戏！
###第二，适配问题。
还是因为原生UI的隔离，再加上android本就是Google的产品。经过初步的开发和调试，Flutter的原生控件在样式上和android原生高度相似，在不同宽高的屏幕中，均可以一套代码适配，适配方式大体和android原生相似，横向和纵向皆可充满屏幕，并按比例放置，可以避免屏幕长宽比变化导致的屏幕溢出。
###第三，切实的解决一套代码大家用的问题。
因为android和iOS切实使用了同一套UI渲染方式，不会像RN那样在开发和迭代过程中遇到必须两个分支才能解决的UI问题。


##五、 Flutter初步入门
1. 写在第一段，Flutter使用了Dart语言。它的整体的UI构建思想和前端和原生移动端是有极大差别的。
一切皆是Widgets。整个页面其实就是由Widgets嵌套组成的控件树，而所有的基础控件，全部继承自Widgets。而且Flutter控件的概念和前端，移动端也有差别。例如一个按钮，移动端一个按钮，会给它设置宽高，位置，颜色，文字。这些属性共同构成了一个按钮。但是在Flutter中并不是这样，颜色本身，就是一个控件，位置本身也是一个控件。当你需要一个按钮出现在正确的位置，长得正确，并且做正确的事，你就需要嵌套4,5层的不同控件，每一层的控件，都会将界面效果本身以child:Widgets()的形式传递下去。以下是一个示例代码：

   ```objc 
Padding(
      padding: const EdgeInsets.all(2.0),
      child: SizeTransition(
        axis: Axis.vertical,
        sizeFactor: animation,
        child: GestureDetector(
          behavior: HitTestBehavior.opaque,
          onTap: onTap,
          child: SizedBox(
            height: 128.0,
            child: Card(
              color: Colors.primaries[item % Colors.primaries.length],
              child: Center(
                child: Text('Item $item', style: textStyle),
              ),
            ),
          ),
        ),
      ),
);
```
大家可以看到，Padding，SizeTransition，GestureDetector，Center这些属性控件本身是和Card卡片控件这种传统控件同一级别的存在。而控件树的嵌套，构成了整个页面的布局。

2. 控件本身分为StatelessWidget 和StatefulWidget 两种状态。从下图中我们也能看到，StatefulWidget 的功能是帮助界面进行动态展示，而StatelessWidget 则是一个静态展示的效果。因此我们也就可以简单的理解，StatefulWidget 的控件可以改变自身的状态，而StatelessWidget 不能改变自身的状态，类似一个final的效果。但是不同的是，StatelessWidget 的控件可以通过嵌套，让包裹着它的父Widget控件来改变它的状态。
  ![image](https://github.com/DZby1990/Flutter_survey_report/tree/master/pic/4.png "")

3. Flutter工程结构。

 
  ![image](https://github.com/DZby1990/Flutter_survey_report/tree/master/pic/5.png "")
Flutter本身使用了androidStudio进行开发，lib为工程代码，pubspec.yaml为配置文件，android及ios为原生代码，images里为图片内容。

4. 目前已经使用Flutter的团队
美团，闲鱼，腾讯直播，github等。

##六、 本次调研的目标

###1. 仿照体验中心完成部分页面，与原生对比。
图一、flutter页面   

  ![image](https://github.com/DZby1990/Flutter_survey_report/tree/master/pic/6.png "")   
 图二、原生页面
 
  ![image](https://github.com/DZby1990/Flutter_survey_report/tree/master/pic/7.png "")
     
Flutter和原生，在样式和交互感受上，可以达到无感知。

###2. 研究其与原生混合开发的可行性与大致性能。

Google官方文档本身建议不进行混合开发。因为其页面栈与原生完全分离，如果使用大量FlutterActivity混合原生界面，每个FlutterActivity启动皆会消耗额外资源，导致加载速度变慢。
目前闲鱼已经推出了一个混合开发框架，其基本思路是复用FlutterActivity，通过反射替换其实例，达到复用的效果，使其在启动时减少额外的资源消耗。
而原生与Flutter的交互依靠MethodChannel进行，目前已尝试调通了Flutter通过MethodChannel获取Android设备信息的功能。

###3. 应用场景问题

目前Flutter主要应用在Android端及iOS端。官方已给出了Web端的示例工程以及2d游戏的示例工程。

##七、 目前发现的问题

1. 使用了Flutter后的应用，应用包大小会明显高于原生app。
2. Dart语言及Flutter上手成本较高，需要逐渐熟练。目前本人初学Flutter绘制一个简单体验中心页面，所需要时间大致是原生的三倍。
3. 键盘点击挡住输入框的问题。这个问题在初步开发时遇到，目前未解决。（隐射出目前非大厂使用人数较少，一些原生的简单小问题需要花费时间解决。）
4. 第三方库较少，很多需要自己造轮子。
5. Dart语言反射极不友好，目前没有成熟的数据库ORM框架。
6. 目前一些原生的功能性sdk并没有flutter端，因此存在被动需要混合界面栈的可能性。
7. 虽然官方说Flutter的开发工具可以是轻量级的，但是由于Dart代码和Widgets控件树有大量括号，阅读依赖AndroidStudio的自动注释功能来知晓应该把Widgets括号括在哪


## 关于作者

郑逸钧，来自某支付公司，资深android开发。个人主页：http://engineerblog.cn
