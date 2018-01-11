---
title: css3过渡
date: 2017-03-20 13:22:31
tags: [css3]
categories: css
---
## 过渡transition

混合写法:
transition:过度的属性 过渡时间,过度的属性 过渡时间...linear(匀速) 延迟时间;

transition: all 2s linear 2s;

transition: width 1s,background-color 1s;

过渡必须加给盒子本身

拆分写法:
过渡属性
transition-property:width;
过渡时间
transition-duration:2s;
过渡曲线(linear线性 ease-in加速 ease-out减速 ease-in-out先加速后减速)
transition-timing-function:linear;
过渡延迟执行时间
transition-delay:2s;

<!-- more -->

案例:小米产品页

感想:css3的过渡属性十分简便,只用写好原始状态和最终状态通过过渡连接即可,但是也有局限性,只能对自己产生影响,这是硬伤,但是其快捷的用法较js方便许多,即使css3能做的js大部分都能做到(小部分如背景颜色,背景阴影做不到 ).
以下的效果,鼠标移上时出现span(位移改变),出现阴影,item上移.  

还可以用js来完成  

1. animate函数来移动位置  
  
2. 亦或是slide show or hide , fade等函数让其在原地出现或是消失  

但是box-shadow还是无法让js实现.   

那什么时候用css3过渡什么时候用js动画呢?看动作是作用于自身还是要作用于别处(例如小米手风琴只能改变自身盒子宽度,无法缩减别的盒子宽度)

		body{
			background-color: #eee;
		}
		.items{
			width: 1100px;
			margin: 100px auto;
		}
		.item{
			margin-left: 20px;
			float: left;
			width: 200px;
			height: 300px;
			background-color: #fff;
			text-align: center;
			position: relative;
			overflow: hidden;
			transition: all 0.7s;

		}
		.item img{
			width: 200px;
		}
		.item span{
			position: absolute;
			bottom: -80px;
			left: 0;
			width: 100%;
			height: 80px;
			background-color: #f40;
			transition: all 0.5s;
		}
		.item:hover{
			box-shadow: 4px 4px 10px 10px #ccc;
			margin-top:-5px;
		}
		.item:hover span{
			bottom: 0;
		}

