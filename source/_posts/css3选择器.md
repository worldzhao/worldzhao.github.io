---
title: css3选择器
date: 2017-03-17
tags: [css3]
categories: css
---

特点：选择器更灵活 行为表现丰富

**属性选择器**   
符号 [] 通过标签属性来选择  
语法:E[attr="value"]

	E:[title] 选中页面中的E元素，并且E需要带有title属性
	^:开头 div[title^="aa"]/*带有title属性,且值为以"aa"开头的div*/
	$:结尾 div[title$="bb"]/*带有title属性,且值为以"bb"结尾的div*/	
	*:包含 div[title*="aa"]/*带有title属性,且值包含"aa"的div*/
	~:包含div[title$="bb"]带有title属性,属性值列表中(以空格分割),且值为以"bb"结尾的div*/
<!-- more -->

**伪类选择器** 
符号：冒号   
- 结构伪类：通过结构来筛选

		li:first-child    /*选中第1个li:选中li父元素ul的第一个子元素!!!并且这个元素是li标签 相当于li:nth-child(1)*/
		li:last-child    /*选中最后1个li:选中li父元素ul的最后一个子元素，如果是li则追加样式如果不是则样式失效 相当于li:nth-last-child(1)*/
		li:nth-child(11)  /*选中第11个li:选中li父元素ul的第11个子元素.索引从1开始*/
		li:nth-child(odd) /*选中奇数*/
		li:nth-child(even)/*选中偶数*/	
		li:nth-child(2n)  /*此处表示选中偶数*/
		li:nth-child(-n+5)/*选中前五个-0+5 -1+5 -2+5 -3+5 -4+5 -5+5*/
		li:nth-last-child(-n+5) /*选中后五个*/
		li:nth-child(7n)  /*选中7的倍数个*/
	
	    注: p:nth-of-type(2)//找到p元素的父级下的第二个p元素
            p:nth-last-of-type(2)//找到p元素的父级下的第二个p元素
            p:first-of-type
            p:last-of-type

n表示0.1.2.3.4.5.6.7.8...
偶数：2n even 奇数：2n-1 odd 前五个:-n+5

- empty伪类选择器
div:empty 选中内部为空的div元素

- target锚点选择器
h2:target
配合锚点使用，对应锚点被激活触发样式.
**可以用来实现选项卡，当选项链接被点击时通过锚点样式使得对应内容显示出来，无需使用Js.**

* 针对表单有专门的伪类选择器，如  
 
		input:enable{}//选择可以编辑的输入框
		input:disable{}//选择不可编辑的输入框
	    input:checked{}//选择被选中的checkbox或radio,可以用于模拟单选框

* not选择符
 

        h1:not(.a){}//对h1所有标签添加样式，class='a'的元素除外
**伪元素**
before与after  
用CSS模拟出html效果，假的标签  
伪 元素 伪：假的 ，元素：标签  
标志性符号：双冒号(::)
css  

	span::before{
		content:"今天";
	}
	span::after{
		content:"真好";
	}
    
html
  
    <span>今天</span>


dom  
    `<span>::before 今天 ::after</span>`  
必须要有content属性，如果为空则为""  

**伪元素选择器**  
<pre>
span::first-letter /*选中第一个字*/  
p::first-line /*选中第一行*/  
p::selection /*表示当前选中的区域，通常改变选中区的背景颜色与当前颜色*/  
</pre>  

首字下沉效果：
<pre>
p:first-child::first-letter{
    font-size:40px;
    float:left;
}
</pre>


选中区颜色:

	p::selection{
		color:red;
		background-color:green;
	}

transition:渐变时间

	.box{
		width: 100px;
		height: 100px;
		background-color: red;
	}
	.box:hover{
		width: 200px;
		height: 200px;
		background-color: blue;
		transition:1s;//过渡时间1s
	}



tips:外部css的地址引用是相对于该css文件的 ， 而内部css的地址引用是相对于html文档的

