# Effective Object-C 52：23-28
### 四、协议与分类

Object-C 语言特性： “协议” （protocol）、“分类”（Category）

#### 23.通过委托与数据源协议进行对象间通信

* 委托模式为对象提供一套接口，对象间事件通知方法之一。
* 将委托对象支持接口定义的协议，在协议中把需要处理的事件定义成方法。
* 若有必要，可以实现有位段的结构体，将委托对象是否能响应相关协议这个信息缓存。

委托模式：定义一套接口，对象若想接受另一个对象的委托，则需要遵从此接口。 委托对象（delegate）
数据源模式：数据源（Data Source）流向类（Class） 

![网络请求整体流程图](https://github.com/RocAndTrees/objective-C52/raw/master/resource/image/objec-c52/23-2两个模式.png)

例子网络请求整体流程图：

![网络请求整体流程图](https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/23-1回调委托对象流程.png)


本对象和委托对象之间的所有权关系：

![本对象和委托对象之间的所有权关系图](https://raw.githubusercontent.com/RocAndTrees/objective-C52/master/resource/image/objec-c52/23-3所有权关系图.png)


协议定义：

```
@protocol EOCNetWorkFetcherDelegate 

@optional
- (void)netWorkFetcher:(id)fetcher
        didReceiveData:(NSData*)data;
- (void)netWorkFetcher:(id)fetcher
        didFailWithError:(NSData*)data;
- (void)netWorkFetcher:(id)fetcher
        didUpdataProgressTo:(NSData*)data;

@end

```
此类的接口定义：

```
@interface EOCNetWorkFetcher : NSObject

@property (nonatomic, weak) id<EOCNetWorkFetcherDelegate> deleage;

@end

```
委托对象调用可选方法(+判断委托对象是否响应相关选择子)：

```
if ([_deleage respondsToSelector:@selector(netWorkFetcher:didReceiveData:)]) {
        [_deleage netWorkFetcher:self didReceiveData:data];
    }

```

如果需要经常使用响应特定的选择子，频繁的执行检察工作没有必要，可以通过结构体、位段（bitfield）数据类型 来缓冲判断：

```
struct data {
    unsigned int didReceiveData : 1;
    unsigned int didFailWithError : 1;
    unsigned int didUpdataProgressTo : 1;
 } _deleageFlags;
// set flag
_delegateFlags.didReceiveData = 1;   
//_deleageFlags.didReceiveData = [_deleage respondsToSelector:@selector(netWorkFetcher:didReceiveData:)];
// Check flag
if (_delegateFlags.didReceiveData){
	//Yes , flag set
}

```

#### 24.将类的实现代码分散到便于管理的数个分类之中

* 使用分类机制把类的实现代码划分成易于管理的小块
* 将“私有”的方法归入名为 Private的分类中，以隐藏实现细节。

例子： 

```
@interface EOCPerson （Friendship)
-(void)addFriend:(EOCPerson*)person;
-(void)removeFriend:(EOCPerson*)person;
-(void)isFriendWith:(EOCPerson*)person;
@end

@interface EOCPerson （Work)
-(void)performDaysWork;
-(void)takeVacationFromWork;
@end

@interface EOCPerson （Play)
-(void)goToTheCinama;
-(void)goToSportsGame;
@end



@implementation EOCPerson(Friendship) 
-(void)addFriend:(EOCPerson*)person{
	/* ... */
}
-(void)removeFriend:(EOCPerson*)person{
	/* ... */
}
-(void)isFriendWith:(EOCPerson*)person{
	/* ... */
}

@end
```

#### 25.总是为第三方类的分类名称加上前缀

* 向第三方类中添加分类时，总应给其名称加上你专用的前缀
* 向第三方类中添加分类时，总应给其中的方法名加上你专用的前缀

>问题：如果类中本来就实现一个方法、分类中也实现会覆盖“主实现”中的相关方法，而如果添加了多个分类，其中有两个分类有同名的方法，加载时机晚的那个分类会覆盖前一个分类的方法，结果造成执行的结果可能就跟预期不同。
>
>解决方案：以命名空间来区别个个分类的名称与其中定义的方法。

例子：

```
@interface NSString (ABC_HTTP)

-(NSString *)abc_urlEncodeString;

-(NSString *)abc_urlDecodedString;

@end
``` 

#### 26.勿在分类中声明属性

* 把封装数据所用的全部属性定义在主接口中。
* “Class-continuation 分类” 之外的其他分类中，可以定义存取方法，但尽量不要定义属性。

1. 分类定义属性编译器无法自动合成该属性的存取方法，需要手动实现。
2. 多次实现存取，容易造成代码冗余，内存管理上也会出现问题。

#### 27.使用“calss-continuation 分类” 隐藏实现细节
* 通过“class-continuation 分类” 向类中新增实力变量
* .h 设置成只读的属性，若需要可以在“class-continuation 分类”重新设置成“可读写”
* 私有方法的原型声明在“class-continuation 分类”里面。
* 隐密的协议可以写在“class-continuation 分类”中。

什么是“class-continuation 分类”：
比如类EOCPerson  写在 .m 内

```
@interface EPCPerson()
//Methods here
@end

```

#### 28.通过协议提供匿名对象

* 协议可在某种程度上提供匿名类型。具体的对象类型可以淡化成尊从某协议的id类型，协议里规定了对象所应该实现的方法。
* 使用匿名对象来隐藏类型名称（或类名）
* 如果具体类型不重要，重要的是对象能够响应（定义在协议里的）特定方法，那么可以用匿名对象来表示。

匿名对象（anonymous object）：其他语言：内联形式创建出来的无名类 ；object-c：只要遵从协议的任何对象。
比如 ： `@property(nonatomic, weak) id<EOCDelegate> delegate` delegate 就是个匿名对象。 
`id <protocol>`

<!--处理数据库连接（database connection）-->








