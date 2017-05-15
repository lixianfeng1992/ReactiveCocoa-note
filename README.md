# ReactiveCocoa-note

## RACCommend
```
//RACCommend 处理事件，把事件数据都包装在累中处理，很方便的监听事件是否执行完成  不能返回一个空的信号
RACCommand *commend = [[RACCommand alloc]initWithSignalBlock:^RACSignal *(id input) {
    NSLog(@"%@",input); //只要执行命令就会调用该方法。
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        //发送数据
        [subscriber sendNext:@"执行命令产生的数据"];
        //发送完成
        [subscriber sendCompleted];
        return nil;
    }];
}];

//监听事件是否完成  当前命令内部发送数据完成，一定要主动发送完成
[commend.executing subscribeNext:^(id x) {
    if ([x boolValue]== YES) {
         NSLog(@"当前正在执行");
    }else{
        NSLog(@"执行完成／还没有执行");
    }
}];

//executionSignals:信号源，这里x依然是一个信号，如果想获取信号中的内容可以降阶
[commend.executionSignals subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];

//获取最新发送的信号， switchToLatest就是降阶方法中的一个，类似还有flatten，concat
[commend.executionSignals.switchToLatest subscribeNext:^(id x) {
     NSLog(@"%@",x);
}];

//触发commend
[commend execute:@(1)];
```