---
layout: post
title: "让内联（inline）元素自动换行"
date: 2012-07-18 15:09
comments: true
categories: 工作点滴 短文
---
有时候我们需要让一个内联元素在新行显示，方法无非这样几种：

1. **直接对该元素 `display:block`**  
这样做的问题在于，如果你要处理的元素是链接，那么这整行都变得可以点击了。  
2. **在元素外套上一层 `div`**  
嗯，基本上满足大部分需要。
3. **不增加标签，直接用css实现。**  
因为新老系统要做兼容，如果采用方法2，则需要在代码中新增模板来区分对待，如果能通过css选择器区分不同情况下的a标签如何显示当然是最好了的，其实只通过css也可以直接做到:  
```css 
.xx-wrap a:before{
    content: '\a';
    white-space: pre;
}
```
解释一下，斜杠A（\a）表示unicode中的换行符，换行符属于“空白符”（white-space）的一种，`white-space`属性用于控制空白符如何显示，`white-space: pre`则意味着换行符的显示方式和在`pre`标签中一致（即该怎样显示，就怎样显示），关于`white-space`的其他用法可以参考[这里](http://www.w3school.com.cn/css/pr_text_white-space.asp)。整个表达式的意思是在a的前面隐式加上一个换行。

*更多内容可以参考[Whitespace and generated content](http://cheeaun.com/blog/2005/06/whitespace-and-generated-content)*