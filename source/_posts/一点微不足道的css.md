---
title: 一定微不足道的css
date: 2017-10-22
tags: [css]
categories: css
---
最近被CSS怼成傻逼，总结一下平时没有注意的点，持续更新。

* 问题1：一个没有设置宽度的div，它的宽度是多少？
* 问题2：把问题1里面的div设置为float为left呢？
* 问题3：float脱离了文档流吗？
* 问题4：脱离了文档流文字为什么还会环绕呢？
* 问题5：postion:absolute脱离了文档流吗？
* 问题6：postion:absolute没有设置left与top，div位置在哪，宽度如何？
* 问题7：与absolute搭配的往往有z-index这个属性。那么如果有一个父元素z-index为1000，子元素z-index为100，谁在上面？

### 问题1：一个没有设置宽度的div，它的宽度是多少？

回答一：div为块级元素，每个块级元素默认占一行,自动充满父级元素的内容区域。

>W3C:div元素没有特定的含义。除此之外，由于它属于块级元素，浏览器会在其前后显示折行。

### 问题2：把问题1里面的div设置为float为left呢？

回答二：设置了float:left后，如果没有设置宽度，宽度是被内容撑开的，如果设置了宽度就是该宽度。

>W3C:如果浮动非替换元素，则要指定一个明确的宽度；否则，它们会尽可能地窄。
注：替换元素是浏览器根据其标签的元素与属性来判断显示具体的内容，如input/img/select等等。

### 问题3：float脱离了文档流吗？
### 问题4：脱离了文档流文字为什么还会环绕呢？

回答三：脱离了文档流，但仍然保持着部分的流动性。
回答四：同上。

>MDN:float CSS属性指定一个元素应沿其容器的左侧或右侧放置，允许文本和内联元素环绕它。该元素从网页的正常流动中移除，尽管仍然保持部分的流动性（与绝对定位相反）。

### 问题5：postion:absolute脱离了文档流吗？

回答五：脱离了文档流

>MDN:绝对定位的元素不再存在于正常文档布局流程中.相反，它坐在它自己的层独立于一切。

### 问题6：postion:absolute没有设置left与top，div位置在哪，宽度如何？

回答六：如果没有设置left,top又没有设置right，bottom，它跟static时的位置一样。也就是说，如果其前面还有一个div，他就在这个div后面，和static时一样，不会浮在上面，也不会在左上角（当然，如果前面没有元素就在左上角了）。

测试：
```css
.wrapper{
    width: 500px;
    height: 500px;
    background-color: grey;
    position: relative;
}
.upper{
    height:100px;
    background-color: yellow;
}
.inner{
    width:200px;
    height:200px;
    background-color: red;
}
```

```html
<div class="wrapper">
        <div class="upper"></div>
        <div class="inner"></div>
    </div>
```

结果如图：

![image.png](http://upload-images.jianshu.io/upload_images/4869616-616b367cbfdfbe8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们再来给inner加上position: absolute;

```css
.inner{
    width:200px;
    height:200px;
    background-color: red;
}
```

结果如图：

![image.png](http://upload-images.jianshu.io/upload_images/4869616-616b367cbfdfbe8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可以看出，设置了postion:absolute后，inner并没有如我们预想的那样和left:0 top:0一般蹲到左上角，而是乖乖的呆在了upper后面，和之前static一样。

### 问题7：与absolute搭配的往往有z-index这个属性。那么如果有一个父元素z-index为1000，子元素z-index为100，谁在上面？

回答七：子元素在上面。
先看代码：

```html
<div class="par">
    <div class="child"></div>
</div>
```

```css
.par{
    position: absolute;
    width:300px;
    height:300px;
    z-index:1000;
    background-color: #000;
}
.child{
    position: absolute;
    width:150px;
    height:150px;
    z-index:500;
    background-color: #fff;
}
```

效果如下：

![image.png](http://upload-images.jianshu.io/upload_images/4869616-f73e059ceb54e7a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

子元素在上面，即使父元素的z-index远远大于子元素的z-index

以下转自[segmentfault](https://segmentfault.com/q/1010000009843297)
>根据规范，z-index是应用到定位元素的，也就是position属性不为relative的元素，否则，设置z-index是没有意义的；
z-index的作用有两点，一是设置在当前堆叠上下文(stacking context)中的层级；二是创建一个新的堆叠上下文；
z-index并不是设置的值越高，就会越靠近用户，还和堆叠上下文有关系；

1. 在同一个堆叠上下文中的元素，z-index越高越靠近用户；
2. 在不同堆叠上下文中的元素，如果堆叠上下文一距离用户更近，那么它的所有子元素都在另一个堆叠上下文子元素的前面，也就是离用户更近，不同堆叠上下文中的子元素不可能发生交叉；
3. 所以，z-index其实不是一个绝对值，而是一个相对值；