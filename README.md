# RAC之响应式编程
####开篇扯淡
######在项目开发过程当中，一般我们看到生活中网页当中更多响应式编程的栗子。响应式布局，自动校验email，自动校验数据等等都是响应式编程的一部分。小编最近刚刚接AngularJS.里面自动校验的栗子如下
```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script src="http://apps.bdimg.com/libs/angular.js/1.4.6/angular.min.js"></script> 
</head>
<body>

<div data-ng-app="" data-ng-init="quantity=1;price=5">

<h2>价格计算器</h2>

数量: <input type="number" ng-model="quantity">
价格: <input type="number" ng-model="price">

<p><b>总价:</b> {{quantity * price}}</p>

</div>

</body>
</html>
```
那么IOS开发当中当然也有类似的框架来做。小编暂且讲一下[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)屌炸天的一个框架。顺便说一下用处吧。
####ReactiveCocoa原理
ReactiveCocoa其实总得来说是信号机制，一般我们在做项目开发当中，会处理n多事件响应，简单的按钮点击，视图滑动，网络请求，视图刷新等等，一般我们在处理这些事件的时候，往往采用的开发模式是Delegate,Notification,Block,KVO等，单单小项目还Ok.一旦项目变大。会发现你的项目状态变量处理也越来越多。


