# ReactiveCocoa 5.0-note
## 属性
MutableProperty 允许我们监听其值的变化
```
var property = MutableProperty<Int>(100)
```
我们可以通过改变其 value 来改变属性的值，并通过其 producer 来监听值的变化
```
property.value = 200
        
property.producer.startWithValues { (value) in
    print(value)
}

property.value = 300

// 输出 
// 200.0  
// 300.0
```


## 绑定
 <~ 运算符是提供了几种不同的绑定属性的方式。注意这里绑定的属性必须是 MutablePropertyType 类型。
- property <~ signal 将一个属性和信号绑定在一起，属性的值会根据信号送过来的值刷新。
```
label.reactive.text <~ textField.reactive.continuousTextValues
```
- property <~ producer 会启动这个producer，并且属性的值也会随着这个产生的信号送过来的值刷新。
```
button.reactive.isEnabled <~ signalProducerA.combineLatest(with: signalProducerB).map({ (name, phone) -> Bool in
    return name.characters.count >= 3 && phone.characters.count == 11
})
```
- property <~ otherProperty 将一个属性和另一个属性绑定在一起，这样这个属性的值会随着源属性的值变化而变化。

## Action
```
// 创建 Action, 可处理异步点击事件
var action = Action<(), Bool, NoError> { _ in
    return SignalProducer { observer, disposable in
        // 处理事件
        observer.send(value: true)
        observer.sendCompleted()
    }
}

// button 点击事件与 action 绑定
button.reactive.pressed = CocoaAction(action)

// 监听 action 结果
action.values.observeValues { (value) in
    print(value)
}

// 处理 button 点击事件的另一种方式, 可于处理同步点击事件
button.reactive.controlEvents(.touchUpInside).observeValues { (button) in
    print(button)
}
```

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