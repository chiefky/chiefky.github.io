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

### @property (nonatomic,readwrite,copy) NSString *name;

* nonatomic / atomic
* readonly / readwrite
* retain / copy / strong /weak / assign

#### atomic（系统默认）

```
//系统生成的代码如下：
@property (retain) NSString *name;

- (NSString *) name {
    NSString *nam = nil;
    @synchronized(self) {
        nam = [[name retain] autorelease];
    }
    return nam;
}

- (void)setName:(NSString *)name_ {
    @synchronized(self) {
      [name release];
      name = [name_ retain];
    }
}
```

不推荐使用的原因：
atomic的作用只是给getter和setter加了个锁，只能保证代码进入getter或者setter函数内部时是安全的，一旦出了getter和setter，多线程安全只能靠程序员自己保障了。
另外，atomic由于加锁会带来一些性能损耗影响访问速度。所以我们一般推荐声明property为nonatomic，在需要做多线程安全的场景，自己去额外加锁做同步。

#### nonatomic(常用)：

nonatomic跟atomic刚好相反,表示非原子的，可以被多个线程访问。它的效率比atomic快，但不能保证在多线程环境下的安全性，在单线程和明确只有一个线程访问的情况下广泛使用。多个线程可以同时对其进行访问,访问速度快；当一个变量声明为nonatomic时，setter函数会变成下面这样：

```@property(nonatomic, retain) UITextField *userName;

//系统生成的代码如下：
- (UITextField *) userName {
    return userName;
}

- (void) setUserName:(UITextField *)userName_ {
    [userName_ retain];
    [userName release];
    userName = userName_;
}
```

