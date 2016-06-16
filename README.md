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
ReactiveCocoa其实总得来说是信号机制，一般我们在做项目开发当中，会处理n多事件响应，简单的按钮点击，视图滑动，网络请求，视图刷新等等，一般我们在处理这些事件的时候，往往采用的开发模式是Delegate,Notification,Block,KVO等，单单小项目还Ok.一旦项目变大。会发现你的项目状态变量处理也越来越多。项目也越来越复杂。抓比了吧。。。
#####信号&&订阅者
ReactiveCocoa综合了Delegate,Notification,Block,KVO等对于RAC来说。主要是信号和订阅者机制，例如点击一个按钮，产生一个signal，然后被Subscriber订阅后，可以响应相应的事件。[limboy](http://limboy.me/)有个很形象的比喻就是每个signal好比一个插头，Subscriber好比插座，插头可以插在任意的插座上，也就是signal可以被多个Subscriber订阅，但是只有订阅后，才会响应。未被订阅时，为冷信号。订阅之后，成为热信号。

```
传统方式UIButton
UIButton *myButton = [[UIButton alloc] init...];
[myButton addTarget:something action:@selector(myAction) forControlEvents:UIControlEventTouchUpInside];

RAC方式
@property (nonatomic, strong ) RACCommand *commandDelete;
@property (nonatomic, weak) IBOutlet UIButton *deleteButton;

self.deleteButton.rac_command =
    [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        return [RACSignal return:input];
    }];
- (RACCommand *)commandDelete {
    if (!_commandDelete) {
        _commandDelete = self.deleteButton.rac_command;
    }
    return _commandDelete;
}
```
接下来你肯定会骂小编。尼玛。你不是说的很刁很方便嘛。怎么会那么多行。？
哈哈哈
这只是刚起步。。。走起来就会简单很多了。。。
然后就是RAC做动态检查。
这时候应该用到UITextField

```
RAC(self.logInButton, enabled) = [RACSignal
        combineLatest:@[
            self.usernameTextField.rac_textSignal,
            self.passwordTextField.rac_textSignal
        ] reduce:^(NSString *username, NSString *password) {
            return @(username.length > 0 && password.length > 0;
        }];
```
应用环境是，当当前用户名和密码同时输入了数据的时候，用户登录的按钮才可以点击，这时候用到了信号的组合。combineLatest(信号联合)。

```
combineLatest:@[
            self.usernameTextField.rac_textSignal,
            self.passwordTextField.rac_textSignal
        ] 
把usernameTextField的信号和passwordTextField的信号组合起来。
```

```
reduce:^(NSString *username, NSString *password) {
            return @(username.length > 0 && password.length > 0;
        }];
通过上一步产生信号的usernameTextField,passwordTextField获取其中输入的值，然后判断当前的输入是否合法。
```
RAC(self.logInButton, enabled)接受返回值。判断当前按钮是否可以被点击。这样是不是明朗了很多。如果采用常规的做法。需要遵循响应的代理，然后从代理中获取当前的值。每当textField的值改变的时候做一次判断。同时当前textField有两个。需要用tag来区分。这样看起来。是不是简单了很多。


