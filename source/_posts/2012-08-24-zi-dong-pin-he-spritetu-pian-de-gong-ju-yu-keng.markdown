---
layout: post
title: "css sprite 相关工具的相关研究"
date: 2012-08-24 20:54
comments: true
categories: css
---
#导读
**sprite** 图一种常见的css优化手段，但手工完成这项工作难免枯燥，于是有各种大牛发明了各种自动化或者半自动化的工具来解决这一问题。

从要解决的问题来讲，工具们在被制造时有如下几个出发点： 

1. 有一堆小图片，如何生成一张大图和现成的 css。
2. 有一张大图片，如何知道大图片中的小图片对应的 css。
3. 有一堆小图片，以及写好的css，如何生成一张大图和对应的css。

[css sprites](http://cn.spritegen.website-performance.org/)、[gopng](http://alloyteam.github.com/gopng/)、[sprite-factory](https://github.com/jakesgordon/sprite-factory)这几种工具对应出发点1，[spritecow](http://www.spritecow.com/)对应出发点2，[ispriter](https://github.com/iazrael/ispriter)则对应3。 
<!-- more -->
## 应用和原理
#### 有一堆小图片，如何生成一张大图和现成的 css？
**css sprites**似乎最早是腾讯的鬼哥基于C#写的工具版，后来搬到了[网上](http://cn.spritegen.website-performance.org/)，[gopng](http://alloyteam.github.com/gopng/) 则似乎是css sprites的HTML5版继任者*。它们的基本原理是读取图片，然后基于[一些算法](https://github.com/jakesgordon/bin-packing)把小图片拼合为一张大图片，拼合的同时记录小图片在大图片中的位置，以此生成对应的css代码。[sprite-factory](https://github.com/jakesgordon/sprite-factory)则类似，不过因为是开源的命令行工具，相较于前两者而言你可以方便地集成到自己项目的build代码中。

_*没去考证，总之这俩工具都是腾讯的哥哥们搞的_
#### 有一张大图片，如何知道大图片中的小图片对应的 css？
[spritecow](http://www.spritecow.com/)是一个比较半自动化的工具，你如果有一张大图片的话，可以通过spritecow自动定位大图片中的小图片的`position`，并生成该小图片相应的css，简单易用，用用便知，基本原理的话则与上一种做法相反。

#### 有一堆小图片，以及写好的css，如何生成一张大图和对应的css。
如果很不幸，你手头已经有一堆图片，一堆写好的css，以及一堆使用着这些css的HTML代码，显然前述两种做法对你而言只能是做“加法”，而不易全部推倒，还好有[ispriter](https://github.com/iazrael/ispriter)这个工具，这个工具相较于前两者，多了一个css分析步骤，可以在不影响已有css的情况下，生成大图和css，具体的使用和原理可以[看看这篇文章](http://imatlas.com/posts/nodejs-intelligent-merge-css-sprite/)。

##我们的需求
除了文章开始的提到的三种需求，可能更多人会遇到这种困扰（这篇博文就源于此）：

有**一些**不大不小图片，它们都有着已经写好的css sprites代码，随着星转斗移，**一些**不大不小的图片变成了**一堆**，这时候，你想把它们合到一张图片中去，你也不想更改你的css代码和HTML代码，而且最好一个按钮，一切搞定。

**答案是，一键清理，不可能——或者说，极其困难。**

所以，在这种情况下想寻求懒方法的朋友可以关掉浏览器了，再见不送~:P

下面详细分析下产生限制的原因和权衡的解法。

###为什么不可能
好，首先假设你已经认认真真看完了关于ispriter的文章，关于基本原理和用法就差不多明白了，不过多介绍。
如果你已经有一堆，基于这种原理进行自动拼合 sprite 时，会遇到以下限制：

**1. 重复属性定义.  **

以这种常见情况为例：
```css
background:-moz-linear-gradient(top,#adda4d,#86b846);
background:-webkit-gradient(linear,0 0,0 100%,from(#adda4d),to(#86b846));
```
因为`CSSOM`解析器不可能针对某一种前缀来保存解析后的样式表，所以像重复定义但是有多个前缀以适应不同浏览器的样式，`CSSOM`解析器在解析css代码时会覆盖重复的属性定义，解析出的`cssRules`中将只包含最后定义的属性（因为剥离了浏览器，所以也不会识别`-webkit-`这样的前缀），最后经过处理再输出了`cssText`就会“信息不完整”。

so，这意味着，如果要完美实现这个 auto sprite 的功能，得修改 cssom 的解析器让其保存所有重复定义（这违背了 css 的运行时应用规则 - -b），想了想难度， 我决定作罢。

so，在限制下，只能搞搞单独的 `sprite.css` 了  


**2. 写法限制 **

isprite 是处理单张小图片用的，如果要处理多张已经sprite的图片，

考虑这样一种我们*已有*的 sprite 代码的 常见情况：
```css
.icon-another{
  background-image: url("icon1.png");
}
.icon {
  background-image: url("icon2.png");
}
.z-icon-follow {
  width: 8px;
  height: 9px;
  background-position: -88px -137px;
}
.z-icon-fold {
  width: 8px;
  height: 9px;
  background-position: -101px -136px;
}
```
用sprite拼合后的代码类似这样：
```css
.icon-another{
  background-image: url("sprite-another.png");
}
.icon {
  background-image: url("sprite.png");
  background-position: -20px -100px; /* 此时的 `background-position` 依赖于拼合的图片的位置  */
}
.z-icon-follow {
  width: 8px;
  height: 9px;
  background-position: -88px -137px; /* 注意，这里没变！*/
}
.z-icon-fold {
  width: 8px;
  height: 9px;
  background-position: -101px -136px;
}
```
很容易看出问题在于，在解析和处理css规则的时候，我们根本无法得知 `.z-icon-follow` 这条规则究竟会使用哪张图片——配合使用`.icon`还是`.icon-another`？而且，即使
我们已经写了很多的 sprite 代码都采用了这样的写法，[bootstrap](http://twitter.github.com/bootstrap)里的所有 sprite也是这样写的，遵循这样的写法和使用方法，如果真要实现自动sprite，则需要：

- 拟定规则，在处理css的时候对没有`background-image`的css规则应用指定的`background-image`，而且还得根据已有的`background-position`和在最后生成的大图中小图的位置计算出最终icon的`background-image`（好拗口,>_<）。
- 或者扫描HTML文件，知道所有的class是如何使用的（基本上是个不可能任务，因为你的HTML代码很可能是模板生成的）

所以，基本上，呃……(⊙o⊙)…

### 所以没辙了吗？
不是已经说了么，“一键XX”是不可能的。XD。所以真是这样的时候，重构你的代码和图片吧。对于大型或者持续开发的项目，出发点1 所限的开发方式是比较稳妥的。
***
#####别的 sprite 思路？
有个很特别的sprite 方式，是利用data:url，具体做法和相关优缺点请参考下面的文章：
[Data URI的利弊](http://www.cssforest.org/blog/index.php?id=152)