---
layout: post
title: "自定义上传按钮样式 & ajax上传"
date: 2012-07-22 17:11
comments: true
categories: 工作点滴
---
自定义上传按钮这个是个老生常谈的问题了，之前在工作中碰到过，简单记录一下。  
实现基本原理是将`input[file]`表单的透明度设置为0，然后外层用`label`标签覆盖里层`input`，按钮样式写在`lable`当中，如：
```css
.transparent-file{
    opacity: 0;
    -moz-opacity: 0;
    filter:alpha(opacity=0);
    position: absolute;
    width:88px;
    height:32px;
    top:0;
    left:0;
}
.file-lable{
    position: relative; /* 使用定位让lable 和 input 重合 */
}
.button{
    /* 你具体button的样式 */
}
```   
```html
<label for="" class="file-lable button">
    <input type="file" class="transparent-file" name="picture"  />
    上传头像
</label>
``` 
如果你要实现**ajax上传**效果，则可以利用一个隐藏的`iframe`作为上传`form`的`target`：  
```html
<iframe src="about:blank" id="iframe" name="iframe" style="display:none" ></iframe>
<form  id="form-id" action="/upload" enctype="multipart/form-data" target="iframe"  method="post">
    <!-- 上传按钮在这儿 -->
</form>
```
其原理是在那个`iframe`中发起post请求，这样就可以实现ajax上传的效果啦*（注意enctype)*   

如果要实现**选择文件后自动上传**的效果，则需要为file表单绑定`change`事件，当选择文件后会触发`change`事件，此时调用`form`元素的`submit`方法即可：
```js
$file.on("change", function(){
  $form[0].submit()  
}); 
```
**注意!** *之前我有个思路是把`input`隐藏掉，同时在单击另一个button的时候触发这个隐藏`input`的`click`事件，实时证明是可以的，但是却有兼容性问题：*
在IE下，触发了`change`事件后，自动调用是`submit`方法将会报错！


与此同时我写了一个jQuery的小插件：  
<!-- more -->
{% gist 3159053 %}
当然，这是针对公司特定应用的，如果要使用到自己的项目中还得修改一下。用法是：  
```js
$('#form-id').imgUploader(); //option defalt 见源码
```
如果`iframe`有回调则可以在回调中调用`$('#avatar-form').trigger('uploadchange',data);`