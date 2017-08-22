# Effective Object-C 52：29-36
### 五、内存管理

>自动引用计数（Automatic Reference Counting， ARC）

#### 29.理解引用计数

* 引用计数机制通过递增递减的计数器来管理内存。创建成功后是 1， 降为0时对象销毁。
* 对象生命周期中，其余对象通过引用来保留或释放次对象。保留、释放操作对象的计数器递增、递减。

##### 引用计数工作原理
引用计数架构下，对象有个计数器根据计数的字来确认是否释放对象或者保留，Object-C 中叫“保留计数”（retain count）／“引用计数”（reference count）。

* Retain：递增
* release： 递减
* autorelease： 自动释放池（autorelease pool）

<img src="https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/29-1对象生命期中计数量的变化.png" width="300" height="400" />


例子（MRC情况下手动添加释放 retain count）：

```
NSMutableArray *array = [[NSMutableArray alloc] init];
    
    NSNumber *number = [[NSNumber alloc] initWithInt:1000];
    
    [array addObject:number];
    [number release];
    
    //do something with 'array'
    
    [array release];

```

![对象创建释放流程](https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/29-2对象创建释放流程.png)

##### 保留环（retain cycle）
环状互相引用多个对象，导致内存泄漏。

<img src="https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/29-3保留环.png" width="300" height="200" />



解决方案：1. “弱引用”（weak reference）  2.从外界命令循环中的某个对象不再保留另一个对象。



#### 30.以ARC简化引用计数

* 手动“保留”、“释放”->自动化。
* ARC只负责Objective-C对象的内存。

变量：ARC ，先保留新值，再释放旧值，最后设置实例变量。


#### 31.在dealloc方法中释放引用并解除监听

* 在dealloc方法里，应该做的事情就是释放指向其他对象的引用；取消原来订阅的“键值观察”（KVO）／NSNotificationCenter。
* 如果对象持有文件描述符等系统资源，应该专门编写一个方法来释放。  （close）
* 执行异步任务的方法不应在dealloc里调用；正常状态的方法也不应该。

#### 32.编写“异常安全代码”时留意内存管理问题
* 捕获异常时，一定要注意将try块内创立的对象清理干净。
* 在默认情况下，ARC 不生成安全处理异常所需要的清理代码。 开启编译器标志 后，可生成这种代码，不过会导致应用程序变大，并降低运行效率。


#### 33.以弱引用避免保留环

* 将某些引用设为weak，可避免出现“保留环”
* weak 引用可以自动清空，也可以不自动清空。自动清空（autoniling）随ARC引入的新特性，由运行期系统来实现。

#### 34.以“自动释放池块”降低内存峰值

- 略

#### 35.用“僵尸对象”调试内存管理问题

- 略

#### 36.不要使用retainCount

* retainCount 比较鸡肋的功能，无法放映对象生命期的全貌。
* 引入ARC 后，retainCount方法已经正式废止。
