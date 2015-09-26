# NSHipster

---

## [#pragma]((http://nshipster.com/pragma/))

pragma 的原始目的是使源代码适配多种编译器，但是现阶段更多时候用于分割代码，增加可读性。类似用法

```
#pragma mark - UIViewController
- (void)viewDidLoad {
}

#pragma mark - IBAction
- (IBAction)cancel:(id)sender {
}
```

将在代码块跳转的下拉pop中生成横线和名字，便于查找代码

另外 #pragma 多用于临时禁用或者启用一些警告，比如

```
#pragma clang diagnostic push
#pragma clang diagnostic ignored “-Warc-retain-cycles”
    self.completionBlock = ^ {
    
    };
#pragma clang diagnostic pop
```

关于警告，更多可参看[这里](http://onevcat.com/2013/05/talk-about-warning/) 以及[这里](http://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)

另外还有 [Which Clang Warning Is Generating This Message?](http://fuckingclangwarnings.com)

---

## [BOOL / bool / Boolean / NSCFBoolean](http://nshipster.com/bool/)

objc 中一般使用 BOOL 做真假判断。必须注意 BOOL 在 objc 中是被 typedef 为 signed char 的，因此在比较时不建议使用 BOOLValue == YES 或者 NO 之类的写法。另外在返回一个 BOOL 值时，也一定进行条件化后再进行返回。

错误用例

```
static BOOL different (int a, int b) {
    return a - b;
}

if (different(11, 10) == YES) {
  printf (“11 != 10\n”);
} else {
  printf (“11 == 10\n”);
}!
if (different(10, 11) == YES) {
  printf (“10 != 11\n”);
} else {
  printf (“10 == 11\n”);
}!
if (different(512, 256) == YES) {
  printf (“512 != 256\n”);
} else {
  printf (“512 == 256\n”);
}
```

输出为：

```
11 != 10
10 == 11
512 == 256
```

make no sense!

`@(YES)` 得到的是一个 `__NSCFBoolean`，这是 `NSNumber` 的 class cluster 的一个私有类。关于 class cluster 的更多，可以参考 [Apple 文档](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/ClassClusters/ClassClusters.html)

![](http://img.onevcat.com/2013/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202013-11-20%20%E4%B8%8B%E5%8D%886.19.16.png)

---

## [nil / Nil /  NULL / NSNull](http://nshipster.com/nil/)

C 语言

- 对于空原始值 0  
- 对于空指针NULL

Objective-C 在 C 的基础上增加了 

- nil - 空对象
- Nil - 空类和 
- NSNull - 实际的一个对象

任何对 nil 发送的消息都会返回 0 值，objc 很独特的一个点

NSArray 和 NSDictionary 不能存储 nil，因此使用 NSNull 对象来进行表示

Type   | What | Detail
------ | ------------- | ------------
NULL   |  (void *)0    | literal null value for C pointers
nil    | (id)0       |                literal null value for Objective-C objects
Nil    | (Class)0    |            literal null value for Objective-C classes
NSNull | [NSNull null]  |   singleton object used to represent null

iOS 7 SDK中关于这些的定义

```
MacTypes.h
#ifndef NULL
#define NULL    __DARWIN_NULL
#endif /* ! NULL */
#ifndef nil
    #define nil NULL
#endif /* ! nil */

objc.h
define Nil __DARWIN_NULL

_types.h
#ifdef __cplusplus
	#ifdef __GNUG__
		#define __DARWIN_NULL __null
	#else /* ! __GNUG__ */
		#ifdef __LP64__
			#define __DARWIN_NULL (0L)
		#else /* !__LP64__ */
			#define __DARWIN_NULL 0
		#endif /* __LP64__ */
	#endif /* __GNUG__ */
#else /* ! __cplusplus */
	#define __DARWIN_NULL ((void *)0)
#endif /* __cplusplus */
```

---

## [Equality](http://nshipster.com/equality/)

直接对对象进行 == 操作符运算表示比较指针是否相等。默认情况下NSObject的-isEqual:方法也是比较指针相等。

对于特定的NSObject类，一般会存在特定的判等方法，如

Class  | Method 
------ | -------------
NSAttributedString	|	-isEqualToAttributedString:
NSData		|		-isEqualToData:
NSDate			|	-isEqualToDate:
NSDictionary	|		-isEqualToDictionary:
NSHashTable		|	-isEqualToHashTable:
NSIndexSet	|		-isEqualToIndexSet:
NSNumber	|		-isEqualToNumber:
NSOrderedSet|			-isEqualToOrderedSet:
NSSet		|		-isEqualToSet:
NSString	|			-isEqualToString:
NSTimeZone	|		-isEqualToTimeZone:
NSValue		|		-isEqualToValue:

对于自定义的判等，一般我们可以通过实现自己的 `-isEqualXXX:` 方法来比较某些感兴趣的属性。当然也可以重载 `-isEqual:`，但是在这种情况下对于 `-hash` 也进行重载可以有效提高散列效率。

在实现 `isEqual:` 和 `hash` 的重载时，注意：

Object equality is commutative   ([a isEqual:b] ⇒ [b isEqual:a])

If objects are equal, their hash values must also be equal   ([a isEqual:b] ⇒ [a hash] == [b hash])

However, the converse does not hold: two objects need not be equal in order for their hash values to be equal 
([a hash] == [b hash] ¬⇒ [a isEqual:b])

---

## [C Storage Classes](http://nshipster.com/c-storage-classes/)

### auto

C 中默认的存储关键字。自动在程序进入块时为变量申请内存，并且在离开块时进行释放。`auto` 的变量只在块作用域中有效。

### register

在 NS 的世界不常用。和 `auto` 类似，但是变量不会在栈上被申请，而会存储在寄存器中。寄存器比 RAM 快，但是前提是经过良好设计。事实上，如果滥用寄存器进行存储结果往往可能因为不必要地占用寄存器而导致变慢。使用 `register` 实际上只作为编译器的参考，实际上编译器可能不使用寄存器。

### static

这个关键字用于存储时，有两种含义

1. 在方法或函数中的static表示值在不同的call中保持一致。
2. 全局声明的static变量可以被任何方法或函数取到。

在 OC 中，一个常用 `staic` 的地方是单例。objc 最佳的单例实践

```
+ (instancetype)sharedInstance {
	static id _sharedInstance = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		_sharedInstance = [[self alloc] init]; 
	});
	return _sharedInstance;
}
```

线程安全，必定只会被 call 一次

### extern

和 `static` 不同，`extern` 声明的变量或者方法可以在所有文件中可见。全局变量是应该绝对避免的，在 objc 中，`extern` 的典型用法有下面两种：

- 全局字符串常量
```
.h  extern NSString * const kAppErrorDomain;
.m ￼￼NSString * const kAppErrorDomain = @“com.example.yourapp.error”
```
- public 函数 一般用在某些工具类，可以使函数在全局可用

---

## [__attribute__](http://nshipster.com/__attribute__/)

`attribute` 是编译器命令，用来在声明是指定一些的特性，主要用于错误检查，优化和提示等

语法为 `__attribute__` 后紧接两个括号，在里面是逗号分隔的属性。

### 对于 GCC

#### format 格式化输出

```
extern int my_printf (void *my_object, const char *my_format, …) __attribute__((format(printf, 2, 3)));
```

#### nonnull 指定某些输入参数不应该是 NULL

```
extern void *my_memcpy (void *dest, const void *src, size_t len)  __attribute__((nonnull (1, 2)));
```

#### noreturn 

无返回，用在抛出等

#### pure / const

`pure` 用在除了返回以外，对程序没有影响的函数中。返回值只与输入参数以及全局变量有关。编译器将对其进行特殊优化。`const` 更进一步，将只与参数有关（而且参数中不能含有指针），`const` 的函数不应该返回 `void`。

一个例子：

```
int square(int n) __attribute__((const));
```

`pure` 和 `const` 都将会在编译时有大幅优化，比如由于函数的结果只依赖于输入，因此函数可以缓存某一输入的结果，从而再次接受到这个输入是迅速返回。

#### unused

附加在方法上，表示方法很可能是没有被使用的。在这种情况下 GCC 将不会对此方法产生警告。

### 对于LLVM

#### availability

```
void f(void) __attribute__((availability(macosx,introduced=10.4,deprecated=10.6,obsoleted=10.7)));
```

比较常见的可用性 `attribute`，说明版本兼容性。一般可以用 Apple 的宏来代替：

可以参看 ``/usr/include/` 的 `Availability.h` 和 `AvailabilityInternal.h`

#### overloadable

用C来重载C++的方法。比如

```
#include <math.h>
float __attribute__((overloadable)) tgsin(float x)
{ return sinf(x); }
double __attribute__((overloadable)) tgsin(double x)
{ return sin(x); }
long double __attribute__((overloadable)) tgsin(long
double x) { return sinl(x); }
```

---

## [@](http://nshipster.com/at-compiler-directives/)

objc 语法三大特点：方括号，超级长的函数名，@ 符号

### Interface & Implementation

```
@interface…@end
@implementation…@end
```

### Properties

```
@property
@synthesize
@dynamic
```

### Forward Class Declarations

```
@class
```

### Instance Variable Visibility

```
@public
@package
@protected
@private
```

### Protocols

```
@protocol…@end
```

### Requirement Options

```
@required and @optional
```

### Exception Handling

```
@try…@catch…@finally 
```

### Literals

#### Object

Type  | Syntax
------|------
NSString		|	@“” 
NSNumber	|	@42, @3.14, @YES, @‘Z’
NSArray		|	@[]
NSDictionary	|	@{}
Dynamically evaluates |  @()

关于自定义Literals，可以参看[这里](https://github.com/dbachrach/OCUDL)

- Objc

    @selector() ->  -performSelector:withObject:
    
    @protocol() -> -conformsToProtocol:

- C
    
    @encode() 返回一个 type 的 type encoding，这个值用于 encodeValuesOfObjCTypes:
    
    @defs() 返回 objc 类的 layout。 比如定义一个和 NSObject 有相同 field 的类，可以写成

```
struct {
	@defs(NSObject)
}
```
 
现在的 objc runtime 中 `@defs` 已经不可用了

### Optimizations

- @autoreleasepool{} 快速的回收池，在 ARC 时代 NSAutoreleasePool 已经被淘汰
- @synchronized(){} 对某个块在上下文（通常是 self）中做安全执行。用这种方式锁定线程是 expensive 的，对于线程安全来说，推荐使用 NSLock 或者低层级的 locking 方法，比如 `OSAtomicCompareAndSwap32(3)`

### Compatibility

`@compatibility_alias` 允许一个现有的类被 alias 为一个另外的名字，一般用在兼容性中。

---

## [instancetype](http://nshipster.com/instancetype/)

`alloc` 和 `init` 将返回所发送类的实例，这是命名规范。但是对于类的 construtor 方法，同样无法从类型检查中受益。例如

```
[[[NSArray alloc] init] mediaPlaybackAllowsAirPlay]; 
// ! “No visible @interface for `NSArray` declares the selector `mediaPlaybackAllowsAirPlay`”!
[[NSArray array] mediaPlaybackAllowsAirPlay];
// (No error)
```

使用 `instancetype` 关键字，可以针对上下文给出合适的返回类型。

---

## [NS_ENUM & NS_OPTIONS](http://nshipster.com/ns_enum-ns_options/)

在 iOS6 之后，推荐使用 `NS_ENUM` 和 `NS_OPTIONS` 来进行定义。

### NS_ENUM

几种 enum 的对比

最基本的 enum：

```
enum {
    UITableViewCellStyleDefault,
    UITableViewCellStyleValue1,
    UITableViewCellStyleValue2,
    UITableViewCellStyleSubtitle
};
```

指定整数，但是无类型：

```
typedef enum {
    UITableViewCellStyleDefault,
    UITableViewCellStyleValue1,
    UITableViewCellStyleValue2,
    UITableViewCellStyleSubtitle
} UITableViewCellStyle;
```

指定整数以及类型：

```
typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
    UITableViewCellStyleDefault,
    UITableViewCellStyleValue1,
    UITableViewCellStyleValue2,
    UITableViewCellStyleSubtitle
};
```

### NS_OPTIONS

在作为 bitmask 的时候使用，语法和 `NS_ENUM` 一样，但是编译器会为添加组合的警告。

---

## [NSOperation](http://nshipster.com/nsoperation/)

objc 中一般的多线程方式 GCD 和 NSOperation

`NSOperation` 表示一个单一的计算单元。它本身是 abstract 的，为其子类提供了一个有用的线程安全的环境。我们可以直接使用其子类 `NSBlockOperation` 来完成一系列工作。

`NSOperationQueue` 用来组织多线程执行操作，扮演优先队列的角色。要开始某个 `NSOperation`，可以对其发送 `start` 方法，或者将其加入一个 `NSOperationQueue` 中。

`NSOperation` 的状态机转换 `isReady` → `isExecuting` → `isFinished`。一般可以通过 KVO 来对状态转换进行检测。

---

## [NSDataDetector](http://nshipster.com/nsdatadetector/)

`NSDataDetector` 是 `NSRegularExpression` 的子类。用来匹配日期，地址，链接，电话号码和换乘信息等。

```
NSError *error = nil;
NSDataDetector *detector = [NSDataDetector dataDetectorWithTypes:NSTextCheckingTypeAddress | NSTextCheckingTypePhoneNumber error:&error];

NSString *string = @“123 Main St. / (555) 555-5555”;
[detector enumerateMatchesInString:string
                           options:kNilOptions
                             range:NSMakeRange(0, [string length])
                        usingBlock:^(NSTextCheckingResult *result, NSMatchingFlags flags, BOOL *stop)
 {
     NSLog(@“Match: %@“, result);
 }];
```

iOS 版本有一个 `UIDataDetectorTypes`

另外在将自然语言翻译为结构化数据时，可以使用 linguistic 相关API (可以参看[这里](http://objccn.io/issue-7-6/))

---

## [NSCache](http://nshipster.com/nscache/)

`NSCache` 经常被直接用一个 Dictionary 替代，对于 `NSCache` 其实是应该在合适的场景经常使用的

key 不被 copy，因此不需要满足 `NSCopying`

使用方法类似字典，参看[这里](http://stackoverflow.com/questions/5755902/how-to-use-nscache)

```
// Your cache should have a lifetime beyond the method or handful of methods
// that use it. For example, you could make it a field of your application
// delegate, or of your view controller, or something like that. Up to you.
NSCache *myCache = …;
NSAssert(myCache != nil, @“cache object is missing”);

// Try to get the existing object out of the cache, if it’s there.
Widget *myWidget = [myCache objectForKey: @“Important Widget”];
if (!myWidget) {
    // It’s not in the cache yet, or has been removed. We have to
    // create it. Presumably, creation is an expensive operation,
    // which is why we cache the results. If creation is cheap, we
    // probably don’t need to bother caching it. That’s a design
    // decision you’ll have to make yourself.
    myWidget = [[[Widget alloc] initExpensively] autorelease];

    // Put it in the cache. It will stay there as long as the OS
    // has room for it. It may be removed at any time, however,
    // at which point we’ll have to create it again on next use.
    [myCache setObject: myWidget forKey: @“Important Widget”];
}

// myWidget should exist now either way. Use it here.
if (myWidget) {
    [myWidget runOrWhatever];
}
```

---

## [CFBag](http://nshipster.com/cfbag/)

`CFBag` 与 `NSCountedSet` 有类似的功能，但是并不是 toll-free 的，需要特别注意

一般来说 `NSCountedSet`（带有统计次数的set）在实际应用中使用的可能不多，但是其实对于某些场景还是比较有用的。

对于高度定制，可以使用 `CFMutableBag` 和它的各种 callback

```
struct CFBagCallBacks {
   CFIndex version;
   CFBagRetainCallBack retain;
   CFBagReleaseCallBack release;
   CFBagCopyDescriptionCallBack copyDescription;
   CFBagEqualCallBack equal;
   CFBagHashCallBack 
typedef struct CFBagCallBacks CFBagCallBacks;
```

---

## [NSValueTransformer](http://nshipster.com/nsvaluetransformer/)

将一个（类）value 转换为另一类的类，本身是 abstract 的

iOS 中不太使用 `NSValueTransformer` 的原因是没有 binding，并且现有的 block 等已经使胶水代码的数量大大降低

一个简单例子

```
@interface ClassNameTransformer: NSValueTransformer {}
@end

#pragma mark -

@implementation ClassNameTransformer
+ (Class)transformedValueClass {
  return [NSString class];
}

+ (BOOL)allowsReverseTransformation {
    return NO;
}

- (id)transformedValue:(id)value {
    return (value == nil) ? nil : NSStringFromClass([value class]);
}
@end
```

--- 

## [NSExpression](http://nshipster.com/nsexpression/)

`NSPredicate` 其实是由两个 `NSExpression` 的左值和右值两部分构成，中间使用操作符链接（比如 < IN LIKE 之类）

但是大多数开发者使用 `NSPredicate` 的 `+predicateWithFormat:`` 方法来构建 predicate，因此 `NSExpression` 很少用。但是其实它很强力

### 数学计算

```
NSExpression *expression = [NSExpression expressionWithFormat:@“4 + 5 - 2**3”];
id value = [expression expressionValueWithObject:nil  context:nil]; 
// 1
```

一行搞定计算器app

### 函数

```
NSArray *numbers = @[@1, @2, @3, @4, @4, @5, @9, @11];
NSExpression *expression = 
[NSExpression expressionForFunction:@“stddev:” 
                          arguments:@[[NSExpression 
         expressionForConstantValue:numbers]]];
 
id value =  [expression expressionValueWithObject:nil context:nil]; 
// 3.21859…
```

### 统计学

* average:
* sum:
* count:
* min:
* max:
* median:
* mode:
* stddev:

### 基本算数

* add:to:
* from:subtract:
* multiply:by:
* divide:by:
* modulus:by:
* abs:

### 高级算数

* sqrt:
* log:
* ln:
* raise:toPower:
* exp:

### 边界方法

* ceiling
* trunc

### 随机

* random  等同于rand(3)
* random: 从array中选一个随机元素

### 二进制运算

* bitwiseAnd:with:
* bitwiseOr:with:
* bitwiseXor:with:
* leftshift:by:
* rightshift:by:
* onesComplement:

### 日期

* now

### 字符串

* lowercase:
* uppercase:

---

## [NSFileManager](http://nshipster.com/nsfilemanager/)

对于一般需求，可以使用这个 [category](https://github.com/fabiocaccamo/FCFileManager)

很多代码片段示例

* Determining If A File Exists
* Listing All Files In A Directory
* Recursively Enumerating Files In A Directory
* Creating a Directory
* Deleting a File
* Determining the Creation Date of a File
* Moving an Item to Ubiquitous Storage

等等。。

---

## [NSValue](http://nshipster.com/nsvalue/)

作为标量值的 boxing，使用 `NSValue` 将标量值进行封装。一般用于将标量值插入到 collection 中时使用。

使用例子

```
// assume ImaginaryNumber defined:
typedef struct {
    float real;
    float imaginary;
} ImaginaryNumber;
 
 
ImaginaryNumber miNumber;
miNumber.real = 1.1;
miNumber.imaginary = 1.41;
 
NSValue *miValue = [NSValue valueWithBytes: &miNumber
                            withObjCType:@encode(ImaginaryNumber)];
 
ImaginaryNumber miNumber2;
[miValue getValue:&miNumber2];
```
