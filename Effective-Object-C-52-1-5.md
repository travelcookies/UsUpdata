# Effective Object-C 52：1-5

### 一、熟悉Object-C 1-5

Object-C ： 基于C语言基础 + 面向对象特性。

#### 1.了解Object-C 的起源

* OC 是C 的超集。使用动态绑定的消息结构：运行时才会检查对象类型。接受一条消息后，究竟执行何种代码，有运行时环境决定而不是编译器。
* 核心： 运行时机制。 

内存管理：引用计数（reference counting）

重要的是理解 OC 的 <mark>动态绑定的消息结构</mark>。区别于其他函数调用的语言，接受的消息是由运行时环境决定而不是编译器。



```
区别：
1. “编译期” 来完成的语音，性能提升需要重新编译。
2. “运行时组件” 本质是 “动态库”更新运行时组件就可以提升性能。
```

#### 2.在类的头文件中尽量少引用其他文件

* 除非确有必要，否则不要引入头文件。一般：.h 向前声明（@Class ClassName）.m（实现文件）引入头文件。
* 把“该类遵循某协议” 的这条申明 移到“Class-continuation ”分类。 如果不行就把协议单独放着一个头文件中，然后引入。


1. 在.h 里使用 @class   、 尽量 避免在 .h使用#import <> 不然增加编译时间
2. class-continuation category “class-continuation 分类”

#### 3.多用字面量语法，少用与之等价的方法。
* 应该多使用“字面量语法” 创建字符串、数值、数组、字典。
* 应该通过“下标”来访问数组下标或者字典所对应的元素。
* “字面量语法” 创建数组、字典，要确保值内不含nil。

```
相比较于使用比如
NSNumber *someNumber =  [NSNumber numberWithInt: 1];
更推荐使用 ——  会使代码更加整洁
NSNumber *somenNumber = @1;
例子：
    NSString *someString = @"hello world";
    NSNumber *floatNumber = @2.5f;
    NSNumber * doubleNumber= @3.12546;
    NSNumber *boolNumber = @YES;
    NSNumber *charNumber = @'A';
    int x = 5;
    float y = 6.755;
    NSNumber *expressionNumber = @(x * y);
    NSArray *array = @[@"1", @"2", @"3"];
    NSString *number1 = array[0];
    NSDictionary *dictionary = @{@"1":@"123123", @"11":@"123"};
    NSString *dicString = dictionary[@"1"];
```

#### 4.多用类型常量，少用#define 预处理指令

* 不要使用预处理指令定义常量。  1.影响编译器的工作效率。2.有人重定义了常量值编译器也不会报错。
* 实现文件中使用 static const 来定义“只在编译单元可见的常量”。
* 在头文件中使用extern 来声明全局常量。

 ```
 #define ANIMATION_DURATION 03  //不推荐使用
 static const NSTimeInterval kANimationDuration = 0.3; //推荐使用

 ```

#### 5.用枚举表示状态、选项、状态码


* 应该多用枚举来表示 状态、专递方法的选项、状态码的等值，给这些值取些易懂的名字。
* 处理枚举类型的switch语句 不要实现default 分支。这样加入新的枚举值后，编译器会提示开发着：switch有未处理的枚举值。
