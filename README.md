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
然后我们看一下RAC的源文件

```
├── UIActionSheet+RACSignalSupport.h
├── UIActionSheet+RACSignalSupport.m
├── UIAlertView+RACSignalSupport.h
├── UIAlertView+RACSignalSupport.m
├── UIBarButtonItem+RACCommandSupport.h
├── UIBarButtonItem+RACCommandSupport.m
├── UIButton+RACCommandSupport.h
├── UIButton+RACCommandSupport.m
├── UICollectionReusableView+RACSignalSupport.h
├── UICollectionReusableView+RACSignalSupport.m
├── UIControl+RACSignalSupport.h
├── UIControl+RACSignalSupport.m
├── UIControl+RACSignalSupportPrivate.h
├── UIControl+RACSignalSupportPrivate.m
├── UIDatePicker+RACSignalSupport.h
├── UIDatePicker+RACSignalSupport.m
├── UIGestureRecognizer+RACSignalSupport.h
├── UIGestureRecognizer+RACSignalSupport.m
├── UIImagePickerController+RACSignalSupport.h
├── UIImagePickerController+RACSignalSupport.m
├── UIRefreshControl+RACCommandSupport.h
├── UIRefreshControl+RACCommandSupport.m
├── UISegmentedControl+RACSignalSupport.h
├── UISegmentedControl+RACSignalSupport.m
├── UISlider+RACSignalSupport.h
├── UISlider+RACSignalSupport.m
├── UIStepper+RACSignalSupport.h
├── UIStepper+RACSignalSupport.m
├── UISwitch+RACSignalSupport.h
├── UISwitch+RACSignalSupport.m
├── UITableViewCell+RACSignalSupport.h
├── UITableViewCell+RACSignalSupport.m
├── UITableViewHeaderFooterView+RACSignalSupport.h
├── UITableViewHeaderFooterView+RACSignalSupport.m
├── UITextField+RACSignalSupport.h
├── UITextField+RACSignalSupport.m
├── UITextView+RACSignalSupport.h
├── UITextView+RACSignalSupport.m
```
刚刚用到的是UITextField。打开看一下UITextField.

```
- (RACSignal *)rac_textSignal {
	@weakify(self);
	return [[[[[RACSignal
		defer:^{
			@strongify(self);
			return [RACSignal return:self];
		}]
		concat:[self rac_signalForControlEvents:UIControlEventAllEditingEvents]]
		map:^(UITextField *x) {
			return x.text;
		}]
		takeUntil:self.rac_willDeallocSignal]
		setNameWithFormat:@"%@ -rac_textSignal", self.rac_description];
}

```
defer是RACSignal的一个类方法。返回的一个延迟的信号。如果不被订阅，就是冷信号。订阅则成为热信号。

concat连接的是UITextFiled的UIControlEventAllEditingEvents信号。

map是把当前信号映射成为x.text。

takeUntil。看名字大概能看懂，就是在某个事件之前一直获取当前信号。rac_willDeallocSignal。。意思是在UITextField销毁之前一直获取当前输入信号。
这样来分析UITextField，大概就会明朗了很多。
####信号使用场景一。
例如我们在网络上获取一列数据，而后我们需要展示在cell上。what should i do ?
RAC_GET如下

```
@implementation GFHTTPManger (Uniform)

+ (RACSignal *)rac_get:(NSString *)urlString params:(NSDictionary *)params {
    return [[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSURLSessionTask *task = [self GET:urlString parameters:params responseKeys:nil autoRun:YES progress:nil completion:^(BOOL success, id userinfo) {
            [subscriber sendNext:userinfo];
            [subscriber sendCompleted];
        }];
        return [RACDisposable disposableWithBlock:^{
            [task cancel];
        }];
    }] replayLazily];
}
@end
```
```
[self GET:urlString parameters:params responseKeys:nil autoRun:YES progress:nil completion:^(BOOL success, id userinfo)
调用的是 GFHTTPManger 网络管理类下的一个GET请求方法。
```
```
[subscriber sendNext:userinfo];
[subscriber sendCompleted];
将当前信号转化为热信号。发送给订阅者。userinfo是当前用户请求成功之后获取到的数据。做为参数传递给订阅者。
```
然后写好了接口类。如何调用呢。

``
@interface WeiBoContentViewModel()


@property (nonatomic, readwrite) BOOL shouldReloadData;

@property (nonatomic, strong, readwrite) NSArray *weiboArray;

@property (nonatomic, strong, readwrite) NSArray *weiboCommentArray;

@end

@implementation WeiBoContentViewModel

- (void)requestRemoteWeiBo {
    @weakify(self)
    [[[[GFHTTPManger rac_get:@"/weibo/Weiboc/getWeiboList" params:nil]
       filter:^BOOL(NSDictionary *value) {
           NSDictionary *dictTmp = [value objectForKey:@"response"];
           return [[dictTmp objectForKey:@"status"] isEqualToString:@"success"];
       }]
      map:^id(NSDictionary *value) {
          @strongify(self)
          NSDictionary *responseDict = [value objectForKey:@"response"];
          NSDictionary *dataDict = [responseDict objectForKey:@"data"];
          NSArray *notificationArray = [dataDict objectForKey:@"WeiboListInfo"];
          return [self formatData:notificationArray]; //格式化当前json转化为模型数组。
      }]
     subscribeNext:^(NSArray *x) {
         @strongify(self)
         self.weiboArray = [x copy];
         self.shouldReloadData = x.count;
         
     }];
}
```
```
json格式如下
"response": {
        "status": "success",
        "code": 1,
        "data": {
            "WeiboListInfo": [
                {
                    "id": "24",
                    "uid": "-1",
                    "content": "Hello ",
                    "comment_count": "0",
                    "repost_count": "0",
                    "create_time": "1465727950",
                    "WeiboCommentList": {
                        "WeiboCommentListInfo": [
                            {
                                "id": "8",
                                "uid": "-1",
                                "weibo_id": "24",
                                "content": "Hello",
                                "create_time": "1465781699",
                                "status": "1",
                                "to_comment_id": "0"
                            },
                            {
                                "id": "7",
                                "uid": "-1",
                                "weibo_id": "24",
                                "content": "hello",
                                "create_time": "1465727958",
                                "status": "1",
                                "to_comment_id": "0"
                            }
                        ]
                    }
                },
```
filter是过滤当前信号
 return [[dictTmp objectForKey:@"status"] isEqualToString:@"success"];
 当当前status 字段为success时，当前请求成功，否则失败。失败后不再进行下一步。

map 映射当前信号。
网络请求过来的数据是json类型，不过我们需要的是模型数组。然后在map中。我们格式化当前json数据。

映射操作完成之后，subscribeNext执行应设置后的操作。

