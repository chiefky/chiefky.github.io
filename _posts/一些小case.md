## 一些小case
###1. clang warning
在这个区间里忽略一些特定的clang的编译警告

```
 #pragma clang diagnostic push
 #pragma clang diagnostic ignored "-Wgnu"
 //code
 #pragma clang diagnostic pop
```
###2.weak & strong self
````
__weak __typeof(self)weakSelf = self;
self.backgroundTaskIdentifier = [application beginBackgroundTaskWithExpirationHandler:^{
    __strong __typeof(weakSelf)strongSelf = weakSelf;
}]; 
````

weakSelf是为了block不持有self，避免循环引用，而再声明一个strongSelf是因为一旦进入block执行，就不允许self在这个执行过程中释放。block执行完后这个strongSelf会自动释放，没有循环引用问题。

举例：

````
- (void)dismissAction {
    // ....耗时异步操作
    //....
    // 同时dismiss
    [self dismissViewControllerAnimated:YES completion:nil];
}
````

###