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

### NSArray VS NSMutableArray

简单来说，NSArray一经初始化便不可更改，NSMutableArray可以更改




