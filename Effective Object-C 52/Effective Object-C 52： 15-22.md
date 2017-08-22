# Effective Object-C 52：15-22
### 三、接口与API设计

#### 15. 用前缀避免命名空间冲突
* 选择公司、应用程序、其一二之名作为类名的前缀，并在所有代码均使用这一前缀
* 为项目中使用第三方库加上前缀。（一般不用）

OC 没有内置的命名空间（namespace）机制，所以如果命名重名，就会发送命名冲突（naming calsh）项目报错。

重命名符号错误（duplicate symbol error）报错：

```
duplicate symbol _OBJC_METACLASS_$_EOCTheClass in:
	build/something.o
	build/something_else.o
duplicate symbol _OBJC_CLASS_$_EOCTheClass in:
	build/something.o
	build/something_else.o
```

解决方案：

1. 所有名称都加上适量前缀。
2. 使用3字母 != Apple Cocoa（两字母前缀 two_letter prefix）。

#### 16. 提供“全能初始化方法”

* 在类中提供一个全能初始化方法，并在文档中指明。其他初始化方法均调用次方法。
* 若全能初始化方法与超类不同， 则需要覆写超类中对应的方法
* 超类的初始化不适用子类，应该覆写超类的方法，并在其中抛出异常。

 全能初始化（指定初始化方法）（designated initializer）：对象提供必要信息以便其能完成工作的初始化方法。
这个类多个初始化方法，其他初始化方法调用全能初始化。


覆写父类的全能初始化方法并在其中抛出异常：

```
-(id) initWithWidth:(float)width andHeight:(float)height{
    @throw [NSException exceptionWithName:NSInternalInconsistencyException reason:@"Must use initWithDimension: instead." userInfo:nil];
}
```


#### 17. 实现description方法

* 实现description方法返回有意义的字符串，用以描述该实例。
* 调试时候打印：实现debugDescription方法。

在类里添加实现方法 description、 debugDescription

```
-(NSString *)description{
    return [NSString stringWithFormat:@"%@, %@",[self class], self];
}
-(NSString *)debugDescription{
    return [NSString stringWithFormat:@"%@, %@",[self class], self];
}

```


#### 18. 尽量使用不可变对象

* 尽量创建不可变的对象。
* 某些属性仅可用于对象内部修改，则在“calss-continuation 分类” 中将其由readonly属性扩张为readwrite属性。
* 不要把可变的collection作为属性公开，而应提供相关的方法，以此修改对象中可变的collection。

例子： 在类 head 里实现readonly  在“calss-continuation 分类”  readwrite。

```
@interface CustomViewController:UIViewController

@property(nonatomic, copy, readonly) NSString* identifier;

@end

@interface CustomViewController ()

@property(nonatomic, copy, readwrite) NSString* identifier;

@end

```


#### 19. 使用清晰而协调的命名方式

* 遵从标准的Object-C 命名规范。
* 言简意骇，从左至右读起来要像个日常用语（最好）
* 不要使用缩略后的类型
* 风格符合自己的代码或者集成的框架。

方法和命名：

1. 驼峰式大小写命名发（camel casing）：小写字母开头、其后没有单词首字母大写。
2. 类名也用驼峰，不过都要首字母要大写。


##### 方法命名例子：

```
+string
+stringWithString
+localizedStringWithFormat:
-lowercaseString
-intValue
-length
-lengthOfBytesUsingEncoding:
-getCharacters:range:
-(void)getCharacters:(unichar*)buffer range:(NSRange)aRange
-hasPrefix
-isEqualToString:
```

##### 类和协议的命名：

1. 加上前缀。 例如： UIView、UIViewController..
2. 在类名后面加上Delegate 例如：UITableViewDelegate

#### 20. 为私有方法添加前缀

* 私有方法加上前缀区别公有方法
* 不要单用"_" 来做私有方法的前缀，这种做法是预留给apple 公司的。

私有方法添加前缀： "p_"

```
-(void)p_privateMethod{}
```


#### 21. 理解Object-C 错误模型

1. 只在及其罕见的情况下才会抛出异常，程序直接退出。
2. 不那么严重的错误： 方法放回 nil／0， 或是使用NSError 表明有错误发生。

NSError 三条信息：

1. Error domain (错误范围， type ： NSString)
2. Error code (错误码， type： int)
3. user info （用户信息， type： dictionary）

NSError 常见用法

1.由（delegate） 来传递

```
-(void)connection:(NSURLConnection*) connection didFailWithError:(NSError*) error
```

2.由方法的“输出参数” 放回给调用者，不仅能通过返回值判断是否成功，还通过NSError来判定

```
-(BOOL)doSomething:(NSError**)error
NSError *error = nil;
BOOL ret = [object doSomething:&error];
if(error){
	//There was an error
}

BOOL ret = [object doSomething:nil];
if (ret){
	//there was an error
}

```
3.例子

```
-(BOOL)doSomething:(NSError **)error{
	//Do Something that my cause an error
	if(/* there was an error */){
		if (error){
		//Pass the 'error' through the out-parameter
			*error = [NSError errorWithDmain:domain code:code userinfo:userInfo];
		}	
		return NO;
	}else {
		return YES;
	}
	
}
```

#### 22. 理解NSCoping 协议

* 若自定义对象具有拷贝功能，需实现NSCopying 协议
* 根据对象类型 ： 可变、不可变 -> NSCopying 与 NSMuatbleCoping
* 浅拷贝、深拷贝（若需要，则考虑新增一个专门执行深拷贝的方法）， 一般情况应该尽量使用浅拷贝。

两个拷贝协议 NSCopying、NSMutableCopying：

```

- (id)copyWithZone:(nullable NSZone *)zone;

- (id)mutableCopyWithZone:(nullable NSZone *)zone;

```
例子：

```
-(id)copyWithZone:(NSZone *)zone{
     Person* copy = [[[self class] allocWithZone:zone] initWithFirstName:_firstName];
     copy->_friends = [_friends mutableCopy];
    return
}
```

"->" 语法：内部使用实例变量。
