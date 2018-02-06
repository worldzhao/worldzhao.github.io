---
title: 函数节流
date: 2017-09-12
tags: [原生js]
categories: js
---
>以下描述均为函数防抖， 《JavaScript高级程序设计》有误
#### 为什么会有函数节流？

先看一段代码：
{% codeblock lang:javascript %}
    window.onresize = function() {
        var div = document.getElementById("mydiv");
        div.style.height = div.offsetWidth + "px";
    }
{% endcodeblock %}
很明显，监听浏览器窗口的resize事件，并基于该事件改变页面布局。首先，计算offsetWidth属性，如果该元素或者页面上其他元素有非常复杂的CSS样式，那么这个过程将会很复杂。其次，设置某个元素的高度需要对页面进行回流来令改动生效。如果页面有很多元素同时应用了相当数量的CSS的话，这又需要很多运算。

>浏览器中某些计算和处理要比其他的昂贵很多。例如，DOM操作比起非DOM交互需要更多的内存和CPU时间。连续尝试进行过多的DOM相关操作可能会导致浏览器挂起，有时候甚至会崩溃。

很明显，用户如果没事儿干放大缩小浏览器窗口玩儿，那我们监听函数将会不停的被调用，倘若函数过“重”，即假设如上文描述的一般，那么对浏览器的压力将会非常之大，其高频率的更改可能会让浏览器崩溃。

所以我们需要通过某种方式解决这种问题，刚好就是我们的主角——函数节流。

#### 什么是函数节流
大家可能也做过我前面说的没事儿扩大缩小浏览器的无聊事儿，

但是你会发现在某些页面的时候，你*手指按住鼠标左键不动*拖拉扩大或缩小浏览器时，页面内部并没有同步变化，当你*松开手指*时，页面才重新根据浏览器窗口变化。

此处便用到了函数节流的思想。

##### 基本思想
某些代码不可以在没有间断的情况下连续重复执行。第一次调用函数，创建一个定时器，在指定的时间间隔之后运行代码。当第二次调用该函数时，它会清除前一次的定时器并设置另一个。如果前一个定时器已经执行过了，这个操作(清除定时器)就没有任何意义。然而，如果前一个定时器尚未执行，其实就是将其替换为一个新的定时器。目的是只有在执行函数的请求停止了一段时间之后才执行。

##### 基本模式
{% codeblock lang:javascript %}
    var processor = {
      timeoutId: null,

      //实际进行处理的方法
      performProcessing: function() {
        //实际执行代码
      },

      //初始处理调用的方法
      process: function() {
        clearTimeout(this.timeoutId);
        var that = this;
        this.timeoutId = setTimeout(function() {
            that.performProcessing();
        }, 100);
      }
    }
    
    // 尝试开始执行
    processor.process();
{% endcodeblock %}
理解了上面的概念过后再看这个代码还是很清晰的，就不详解了。
为了去操作一个方法创建一个对象感觉怪怪的，可以简化一下。

##### 简化
{% codeblock lang:javascript %}
    function throttle(method,context) {
      clearTimeout(method.tId);
      method.tId = setTimeout(function() {
          method.call(context);
      }, 100);
    }
{% endcodeblock %}
throttle()函数接受两个参数：要执行的函数以及在那个作用域中执行。上面这个函数首先清除之前的任何定时器。定时器ID是存储在函数的tId属性中的，第一次把方法传递给throttle()的时候，这个属性可能并不存在。接下来创建一个新的定时器，并将其ID储存在方法的tId属性中。如果这是第一次对这个方法调用throttle()的话，那么这段代码会创建该属性。定时器代码使用call()来确保方法在适当的环境中执行。如果没有给出第二个参数，那么就在全局作用域内执行该方法。

注：如果有同学对that/context/call等相关语句有疑问，该好好补补课啦，搜索关键字:关于函数调用的this指向 - -!!

##### 解决问题

让我们回到最开始的代码：
{% codeblock lang:javascript %}
    window.onresize = function() {
        var div = document.getElementById("mydiv");
        div.style.height = div.offsetWidth + "px";
    }
{% endcodeblock %}
现在看来这段代码弊端就很明显了，我们需要对这个函数节流一下。让用户拉拉拖拖结束过后再改变样式，节省浏览器资源。
{% codeblock lang:javascript %}
    function throttle(method,context) {
        clearTimeout(method.tId);
        method.tId = setTimeout(function() {
            method.call(context);
        }, 100);
    }

    function resizeDiv() {
         var div = document.getElementById("mydiv");
         div.style.height = div.offsetWidth + "px";
    }

    window.onresize = function() {
            throrrle(resizeDiv);
    }
{% endcodeblock %}
搞定。

只要代码是周期性执行的，都应该使用节流。

参考资料: JavaScript高级程序设计