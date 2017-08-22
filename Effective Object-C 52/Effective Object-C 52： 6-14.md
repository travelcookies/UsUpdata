# Effective Object-C 52：第二章 6-14条
### 二、对象、消息、运行期

#### 6. 理解“属性”这一概念
* 使用@property 语法来定义对象中封装的数据
* 通过“特质” 来制定存储数据所需的正确语义
* 在设置属性所对应的实例变量时，一定要遵从该属性所声明的语义。
* 开发iOS ： 使用nonatomic 属性。atomic 会影响性能。

>   实例变量访问方式： 存取方法（access method）（读取）+ 获取方法（getter）（写入）
> 
> 定义实例：编译器会把其替换成“偏移量”（offset）== "硬编码"（hardcode）
> 编译器计算出来的偏移量，在修改类定义后必须重新编译。
> Object-C 的偏移量存储交给“类对象来处理”
>

@property 语法：
> "点语法" == "存取方法" 
> 
> 自动编写访问属性的方法-> “自动合成” autosynthesis
> 

```
Person* aPerson = [Person new];
aPerson.firstName = @"Bob";//Same as
[aPerson setFirstName:@"Bob"];

NSString* lastName = aPerson.lastName;//Same as
NSString* lastName = [aPerson lastName];

```

> 
> @synthesize ：去除生成变量名的下划线。
> 
> @dynamic 阻止自动合成方法创建  

##### 属性特质
> 原子性（atomic）：某操作具有整体性，系统其他部分无法观察到其中间的操作步骤，只能看到操作前和操作结果。 其他命名：（获取器，设置器，保有）。
> 
> OC 默认情况下 是“原子的”
> 
> 读／写权限
> 1.readwrite ：拥有 getter、 setter 
> 
> 2.readoly ： 仅拥有获取方法。
> 


###### 内存管理语义：

| Name  	    | 语义  |
|:------------|:-----------------------------------------:|
| assign      | “纯量类型”  （scalar type、CGFloat、 NSInterger） |
| unsafe_unretained | == assign 但适用于 ”对象类型“（object type） “非拥有关系” 当对象被摧毁时候，属性值不会清空。   |
| strong      | “拥有关系“ 1.设置新职时：1.保留新值 2.释放旧值 3.把新值设置上去         |
| copy 	    | == strong 但是设置方法不保留新值，而是“拷贝”（copy）。         |
| weak 	    |“非拥有关系” 1.不保留新值 2.不释放旧值。对象遭到摧毁时，属性值也会清空（nil out）。         |



#### 7. 在对象内部尽量直接访问实例变量
`写法： 直接访问实例变量 _属性名 ； 通过属性：self.属性名`

- 对象内部提取，应该通过直接访问实例变量来读，通过属性来写。 <_objc(直接访问) &  self.objc    >
- 在初始化和dealloc方法中，都应该直接通过实例变量来读写数据 <_objc>
- 惰性初始化配置某份数据，需通过属性来读写数据 <self.objc>

#### 8. 理解“对象同等性”这一个概念

* 检测对象的等同性： 1. isEqual： 2. hash
* 相同对象必须具有相同的哈希码， 但两个哈希码相同的对象未必相同。
* 不要盲目逐条检测每条属性， 而是应该按具体需求制定检测方案。
* hash 方法： 使用计算速度快而且哈希码碰撞低的算法。


#### 9. 以“类族模式” 隐藏实现细节

* 类族模式可以实现细节隐藏在一套简单的公共接口后面。
* 系统框架中经常使用类族
* 从类族的公共抽象基类中继承子类时 要小心，应优先阅读开发文档。

工厂模式（Factory pattern） ：基类实现一个”类方法“、子类继承基类 跟根据“类方法” 实现类实例。


#### 10. 在既有类中使用关联对象存放自定义数据

* 使用“关联对象” 机制把两个对象连接起来
* 定义关联对象时可指派对象内存管理语义 == 属性
* 一般情况下不使用“关联对象”， 只有在其他做法不可行时使用，因为这种做法通常会引入难以查找的bug。

对象中存放相关需求：

1. 一般情况下，从对象所属的类中继承一个子类，然后改用这个子类对象。
2. 无法创建子类对象的情况下，使用 “关联对象” （Associated Object）。



“关联对象 ”存储语义：

| 关联类型  	    							|等效的@property属性 |
|:--------------------------------------|:---------------:|
| `OBJC_ASSOCIATION_ASSIGN`      			| assign |
| `OBJC_ASSOCIATION_RETAIN_NONATOMIC`   | nonatomic,retain        |
| `OBJC_ASSOCIATION_COPY_NONATOMIC` 	    | nonatomic,copy        |
| `OBJC_ASSOCIATION_RETAIN` 				| retain  |
| `OBJC_ASSOCIATION_COPY` 	    			| copy        |

方法：

``` 
#import <objc/runtime.h>  

OBJC_EXPORT void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
    OBJC_AVAILABLE(10.6, 3.1, 9.0, 1.0);

OBJC_EXPORT id objc_getAssociatedObject(id object, const void *key)
    OBJC_AVAILABLE(10.6, 3.1, 9.0, 1.0);

OBJC_EXPORT void objc_removeAssociatedObjects(id object)
    OBJC_AVAILABLE(10.6, 3.1, 9.0, 1.0);

```


#### 11. 理解objc_msgSend 的作用

* 消息构成： 接收者、选择子、参数。 给某对象“发送消息”（invoke a messge） == 在该对象上“调用方法”。
* 发送给某对象的消息全部有“动态消息派发系统”（dynamic message despatch system）来处理，该系统会查处对应的方法。


OC、对象的方法调用： “传递消息” （pass a message）。消息：“名称”（name）或者 “选择子”（selecor），可以接受参数，可能有返回值。

C、静态绑定（static biding）编译期就决定了运行时所应调用的函数。

sendMsg 操作流程：

1. 接收着所属类里搜寻“方法列表”(list of methods)
2. 找到与之相匹配的方法名就跳转至其实现代码
3. 找不到 沿着集成体系往上查找，直到找到为止。
4. 还是找不到-> 执行“消息转发”(message forwarding);


#### 12. 理解消息转发机制

* 消息转发机制触发：对象无法响应某个选择子
* 1.动态方法解析 -> 2.转交给其他对象解析 -> 3.完整的消息转发机制  [3个步骤 其中一个步骤完成就不会启动下一个步骤]


消息转发流程：
 
![消息转发流程](https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/12-1消息转发全流程.png)


<!--1.无缝桥接（toll-free bridging）-->



#### 13. 用“方法调试技术” 调试 “黑盒方法”

* 运行期，可以向类中新增或者替换选择子所对应的方法实现
* 通关 “方法调试”向原有实现中添加新的功能。
* 一般调试的时候使用，不宜滥用。


>方法调配（method swizzling）：object-c 对象受到消息后，只有在运行期才能解析。利用这一特性覆写SEL-IMP 关系，就能改变这个类的本身的功能。

动态消息派发系统 根据 ：类的方法列表会把选择子的名称映射到相关的实现上。

IMP： 函数方法指针。  `id (* IMP)(id, SEL, ...)`

类的选择子映射表 图1:

![类的选择子映射表](https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/13-1NSString类的选择子映射表.png)


开发者几个开发方向：1. 新增选择子 2. 改变选择子所对应的方法实现 3. 交换两个选择子所映射到的指针。

改变后映射表 图2:
![改变后映射表](https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/13-2操作后的映射表.png)


```
交换方法：Void method_exchangeImplementations(Method m1, Method m2)
获取相关的方法： Mehod class_getInstanceMethod(Class aClass, SEL aSelector)

Method originalMethod = class_getInstanceMethod([NSString class], @selector(lowercaseString));
Method swappedMethod = class_getInstanceMethod([NSString class], @selector(uppercaseString));
method_exchangeImplementations(originalMethod, swappedMethod);
```

调试方案： 

1. 添加一个分类（category） 一个新方法
2. 与现有的方法互换
3. 可以实现日志、 适用于短时间现有版本方法更改问题。


#### 14. 理解“类对象”的用意

* 没个实例都有一个指向Class对象的指针，用以表明其类型，这些Class对象构成类的继承体系。 [Class isa]
* 使用类型信息查询方法来探知无法在编译器确定类型的对象。
* 尽量使用类型信息查询方法来确定对象类型，不要直接比较类对象，因为某些对象可能实现了消息转发功能。

>"isMemberOfCalss:" 判定对象是否为某个对象的实力
>
>"isKindOfClass:" 对象是否是这个类或者其派生类的实力。

```
//判断对象是否是某个类的实例
if ([objec class] == [EOCSomeClass class]){
	//'objec' is an instance of EOCSomeClass
}
```
