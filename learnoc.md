

# learn oc

## 基本类型

- 整数型

  ```objective-c
  NSInteger(有符号整数)
  NSUInteger(无符号整数)
  //有符号整数与无符号整数都是32位，四字节
  ```

- 字符串

  oc支持C语言的字符与字符串的定义，但是oc使用的是一个字符串类，声明一个字面量的字符串类型：

  ```objective-c
  NSString *string1=@"Hello world";
  NSString *string2=[NSString stringWithFormat:@"%d %s",1,@"String"];
  NSString *string3=[NSString stringWithCString:"A C string" encoding:NSASCIIStringEcoding];
  ```



- NSNumber

  NSNumber将基本数据类型（int, float...）转成类对象，创建方法：

  ```objective-c
  NSNumber *num1=[NSNumber numberWithShort:1];
  ...
  NSNumber *num2=@1;//可以用@赋值
  ...
  ```

- short(2 bytes) long(4bytes)...

- `@`可以将NSInteger,int,NSString等基本类型转化成类对象

- `id`声明的变量为任何继承自NSobject的类对象


## block

- block可以有返回类型和参数，可以当作函数

```objective-c
void (^someBlock)() = ^{
  // Block implementation here
};
```

- block使得oc可以函数式编程，我的理解是用变量存储函数，给函数起了个别名。
- block可以访问定义它范围内的变量，比如:

```objective-c
int additional = 5;
int (^addBlock)(int a, int b) = ^(int a, int b){
  return a + b + addItional;
};

int add = addBlock(2, 5); // < add = 12
```

注意：只可访问不可修改，如果要修改要在变量前面加__block，比如：

```objective-c
- (void)changeValue {
    __block int value = 0;

    void (^someBlock)(void) = ^{
        NSLog(@"value:%i", value); // value:1
        value = 2;
    };
    value = 1;
    someBlock();
    NSLog(@"value:%i", value); // value:2
}
```

- 循环引用

由于block具有引用计数，如果在块的代码用了`self`，就会导致循环引用。（详情见内存管理）比如：

```objective-c
@interface ViewController ()
@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) CompletionHandler handler;
@end

@implementation ViewController

- (void)blocks {
    self.handler = ^{
        NSLog(@"%@", self.name);
    };
}

@end
```



解决办法：可以声明一个`__weak`的局部变量来解决，比如：

```objective-c
@interface ViewController ()
@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) CompletionHandler handler;
@end

@implementation ViewController

- (void)blocks {
    __weak __typeof(self)weakSelf = self;
    self.handler = ^{
        __strong __typeof(weakSelf)strongSelf = weakSelf;
        NSLog(@"%@", strongSelf.name);
    };
}

@end
```



## 类

**继承，封装，多态感觉和C++相差不大**

- OC的类规格说明包含两个部分：定义（interface）和实现（implementation）

  i.定义包含了类声明，变量，方法的定义

​        ii.实现包含了方法的具体代码

- NSObject:NSObject是大部分OC类继承的根类

### 成员函数

**成员定义**

成员定义在类定义最开始的一个花括号中，比如

```objective-c
@interface myobject:NSObject{
  NSString *s;
  NSInteger i;
}
```

**变量作用域**

@package指的是在实现这个类的可执行镜像中可以访问，外部不可访问，其他与C++无异

### 实现方法

比如：

```objective-c
- (instancetype)initWithName:(NSString *)name andAge:(NSInteger)age {
    self = [super init];
    if (self) {
        _name = name;
        _age = age;
    }
    return self;
}
```

- `super`用来调用父类方法

- `self`用来指代对象本身

- `-`代表实例方法，`+`代表类方法，不用类对象就可以调用的方法，类似于C++的静态方法

### NSArray VS NSMutableArray

简单来说，NSArray一经初始化便不可更改，NSMutableArray可以更改

### Property

- 如果定义一个名为name的属性，那么编译器会自动生成_name的成员变量，并且自动添加set和get方法。

定义语法：

```objective-c
@property (attribute1,attribute2) type property_name
```

括号内为property的属性：

- nonatomic /atomic, readwrite/readonly ：原子性/非原子性 ，可读写/只可读

- weak/strong 对于类对象来说（详情见内存管理）

使用属性：

点语法 or get/set函数

```objective-c
person.name=@"sameen";//点语法实现set
NSString *name=person.name;//点语法实现get

[person setName:@"sameen"];//set方法
NSString *name=[person name];//get方法
```

### Category

有时会希望给类增加新的函数又不想继承基类，那么可以用`category`。

声明语法：

```objective-c
@interface classname (categoryname)

@end
  
@implementation classname(categoryname)

@end
```

`classname`是类别为添加函数的类名，`categoryname`为该类别的名字

`notice`:

- 类别方法会覆盖现有方法

- 如果多个类别有同一名字的函数，那么调用哪个函数不确定

- 添加到类的方法会被所有的子类继承

- 可以为一个类添加多个类别

### Extension

扩展也可以不通过继承为类添加新的函数，它与类别的区别是：

- 扩展仅能在原始类的`.m`文件中实现
- 扩展可以为类添加新的属性，类别不行
- 类别是运行时特性，扩展属于编译器特性

### protocol

协议声明语法：

```objective-c
@protocol protocol_name
@required
...
@optional
...
```

协议用于声明独立于任何特定类的方法和属性，如果一个类声明自己满足某个协议，那么这个类必须实现`@required`所有方法。

声明一个类满足某个协议的语法：

```objective-c
@interface MyClass : NSObject <MyProtocol, AnotherProtocol, YetAnotherProtocol>
...
@end
```

## 内存管理

### MRC

手动引用计数有以下操作：

生成一个对象，对象引用计数置为1:`alloc`,`copy`,`new`

持有，对象引用计数加一：`retain`

释放，对象引用计数减一：`release`,`autorelease`

废弃，引用计数降为0应该废弃：`dealloc`

显示，显示一个对象的引用计数：`retain`

`AutoReleasePool`:

- 在线程或事件结束任务之后，`AutoReleasePool`会自动释放其中的变量

解决循环引用：

解决循环引用，将其中一个持有改为赋值即可，`retain`换成`assign`

### ARC

ARC与MRC的区别是保留和释放前者是自动处理，后者是手动处理

解决循环引用问题，ARC用的是`weak`

`__strong`

`__weak`

`__unsafe_unretained`

`__autoreleasing`

在用ARC的时候，不能使用`NSAutoreleasePool` class，应该使用`@autoreleasepool` blocks:

```objective-c
@autoreleasepool {
     // Code, such as a loop that creates a large number of temporary objects.
}
```

### 循环引用

理解循环引用就要理解引用计数的机制，假如A持有B，B也持有A，只有在引用计数降为0的时候，对象才会被释放掉。对于AB双方来说，一直都有对象持有自己，自己一直无法释放。



