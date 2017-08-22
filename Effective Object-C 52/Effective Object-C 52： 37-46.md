# Effective Object-C 52：37-46
### 六、块与大中枢派发

> 块（block） 、大中枢派发（Grand center Dispatch， GCD）
>
>  语法闭包（lexical closure）

#### 37.理解“块”这一概念

* 块石C、C++、Objective-C中的语法闭包
* 可以接受参数、返回值
* 根据范围：“栈块”、“堆块”、“全局块”。分配在栈上的块可以拷贝到堆中，这样的话就和标准的Objective-C对象一样，具备引用计数能力。


##### 基础

块的语法结构：

`return_type (^block_name)(parameters)`

一般应用：
内联块（inline block）直接内联在函数调用中使用。
`numerateObjectsUsingBlock:`


##### 块的内部结构
图1

##### “栈块”、“堆块”、“全局块”。
一开始定义的块其所占的内唇区域是分配在栈内。通过copy 就会分配到堆中。

全局块：不会捕捉任何状态，运行时也无须有状态来参与。所使用的整个内存区域，在编译器已经完全确定，全局块可以声明在全局内存里。


#### 38. 为常用的块类型创建typedef

* 使用typedef重新定义块类型，令块变量用起来更加简单。

因为每个块都具备“固有类型”（inherent type），因此可以赋给适当类型的变量。

```
^(BOOL flag, int value){
    if (flag) {
        return value * 5;
    }else{
        return value * 10;
    }
    
};
  
//变量类型以及赋值语句  
int(^variableName)(BOOL flag, int value) = ^(BOOL flag, int value){
    return 1;
};
    
variableName(YES, 10);

//块类型的语法结构
return_type (^block_name)(parameters)
```

使用C语言中“类型定义”（type definition）特性
例子：

```
typedef int(^EOCSomeBlock)(BooL flag, int value);

EOCSomeBlock block = ^(BOOL flag, int value){
        //Implementation
}

block(YES, 10);
```

#### 39.用handler 块降低代码分散程度

* 创建对象时，可以使用内联的handler 块将相关业务逻辑一并声明。
* 在有多个实例需要监控时：
  1. 委托模式：需要经常根据传入的对象来切换
  2. 块: 可以直接将相关块和对象放在一起。
* 设计API的时候如果用到handler块，可以增加一个参数，通过这个参数来决定把块安排到哪里队列上执行。<可能废除了>



#### 40.用块引用其所属对象时不要出现保留环

* 如果块所捕获的对象直接或间接保留了块本身，那么就得当心保留环问题
* 一定要找个适当的时机解除保留环。`<weak, weakSelf, strongSelf>`



#### 41. 多用派发队列，少用同步锁

- 派发队列可以用来表述同步语义，这种做法要比使用@synchronized块 或者 NSLock 对象更简单
- 将同步与异步结合起来，可以实现与普通加锁机制一样的同步方法，而这么做不会赌赛执行异步派发的线程。
- 使用同步队列栅栏块，可以令同步行为更加高效。



#### 42.多用GCD，少用performSelector

- 略

#### 43.掌握GCD及操作队列的使用时机

- 在解决多线程与任务管理问题时，派发队列并非唯一方案。
- 操作队列提供了一套高层的Objective-C API，能实现纯GCD所具备的绝大部分功能，而且还能完成一些更为复杂的操作，那些操作若改用GCD来实现，则需要另外编写代码。

#### 44.通过Dispatch Group机制，根据系统资源状况来执行任务

- 一系列任务可归入一个dispatch group 中。开发者可以在这组任务执行完毕时获取通知。
- 通过dispatch group 可以在并发式派发队列里同时执行多项任务。GCD会根据系统资源状况来调度这些并发执行的任务。

#### 45.使用dispatch_once来执行只需要运行一次的线程安全代码

- 经常需要编写“只需执行一次的线程安全代码”（单例里）。通过GCD： dispatch_once 函数，很容易就能实现此功能。
- 标记应该声明在staic 或者 global 作用域中，这样的话，在把值需要执行一次的块传给 dispatch_once 函数时，传如的标记也是相同的。


#### 46.不要使用dispatch_get_current_queue

-略
