---
title: IE的那些坑
id: 253
categories:
  - Web
date: 2015-02-25 15:41:35
tags:
---

由于产品需求从IE7开始支持，前段时间花了不少精力填坑，还是备忘一下。

**1. 元素的float属性无效**
- 发现版本：IE7
- 解决方案：将html中具有float属性的元素写在前面。

**2. innerText赋值空格时无效**
- 发现版本：所有IE
- 解决方案：未发现，可以用innerHTML勉强解决。

**3. 元素的attributes和property属性混在一起**
- 发现版本：IE7
- 解决方案：可以使用attr.specified区分，但有特例：给input设置了value时specified属性仍为false（貌似option的value也是如此）。

**4. doucument.createElement创建input后，无法修改name属性**
- 发现版本：IE7
- 解决方案：使用document.createElement(&lt;input name=’xxx’/&gt;)代替。

**5. &lt;input type=’checkbox’ checked=’checked’&gt;, JS获取的attributes里’CHECKED’的值为‘true’**
- 发现版本：IE7
- 解决方案：就用true做比较呗。

**6. &lt;input type=’checkbox’/&gt;调用setAttribute(‘CHECKED’, true/’checked’)无效**
- 发现版本：IE7
- 解决方案：使用obj.checked=true代替，且必须在obj已经放到DOM中才生效，否则报JS异常。

**7. 据说在设置height属性后，超出的部分会溢出，但是实际中没有超过设定的height也溢出了，例如height:100%**
- 发现版本：IE7
- 解决方案：将height属性修改为min-height:100%;

**8. 继上一个问题，div的height:100%，在父元素没设置height属性时无效，例如父元素设置{position:fixed;top:100px;right:0;bottom:0;left:0}时，子元素设置height:100%无效**
- 发现版本：IE7
- 解决方案：给子元素设置\*position:absoulte;

**9. 继上一个问题，如果父元素设置了position:absolute，子元素(实际为间接子元素)在设置margin属性时，有margin-right无效的现象**
- 发现版本：IE7
- 解决方案：设置父元素left:0;top:0;right:0;bottom:0;

**10. z-index必须设置在同级元素才生效，子元素里的z-index再大都不生效**
- 发现版本：IE7
- 解决方案：给同级元素设置: position:relative;z-index:xxx;

**11. 设置position:relavive之后的一个怪异现象：**
```
<div id=’div1’> //relative
    <div id=’div2’>
        <div id=’div3’></div>
        <div id =’div4’></div>
    </div>
    <div id=’div5’></div> //absolute
</div>
```
div1为relative，div5为absolute相对div1进行定位。当给div3设置position:relative后，div5离奇消失
- 发现版本：IE7
- 解决方案：网上搜索说可能对html代码顺序有关，将div5移至div2之前，问题解决。

**12. img元素最好设置height，不然会变形**
- 发现版本：IE10
- 解决方案：未发现。

**13. input的placeholder属性不支持**
- 发现版本：IE9
- 解决方案：模拟。

**14. block元素的display属性设置为inline-block时无效**
- 发现版本：IE7
- 解决方案：先使用display:inline-block属性触发块元素，然后再定义display:inline，让块元素呈递为内联对象（两个display 要先后放在两个CSS声明中才有效果，这是IE的一个经典bug，如果先定义了display:inline-block，然后再将display设回 inline或block，layout不会消失）。

**15. console未定义，未打开调试模式时会报错**
- 发现版本：IE8，高版本模拟IE8时不会出现
- 解决方案：全局定义console。

**16. for…in循环对数组无效**
- 发现版本：IE8，高版本模拟IE8时不会出现
- 解决方案：使用原始的for(var i = 0; i < xx.length; i++)。

**注：**
- 文中的“发现版本”是我出现问题的版本，不排除其他版本也有此问题的可能。
