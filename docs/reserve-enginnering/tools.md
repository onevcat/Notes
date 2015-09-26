## [cycript](http://www.cycript.org)

Cycript allows developers to explore and modify running applications on either iOS or Mac OS X using a hybrid of Objective-C++ and JavaScript syntax through an interactive console that features syntax highlighting and tab completion.

参考文档 [iPhonDev Wiki](http://iphonedevwiki.net/index.php/Cycript)，[Cycript Manual](http://www.cycript.org/manual/)

### 命令以 ? 开头

* ?syntax 高亮
* ?exit
* ?expand 展开 CYON (Cycript Object Notation)。一般用来展开字符串中的 `\n`
* ?debug

### 与 Objective-C 交互

* `@` - @"hello"，@[2,4,5]
* 自省 - `instanceof` `@"hello" instanceof String` -> `true`
* selector - @selector(this:is:a:message:) -> sel_registerName("this:is:a:message:")
    可以直接 call() 一个 selector
* `#` 代表 object：UIApp -> #"<SpringBoard: 0x10e803490>", s = #0x10e803490 -> #"<SpringBoard: 0x10e803490>"
* 使用 `*` 返回对象结构的 representation，使用 `->` 访问成员
* `messages` 获取类型能响应的消息：`NSObject.messages`
* frame -  [[1, 1],{width:2,height:2}]
* 添加 Category
```
cy# @implementation NSObject (MyCategory)
cy> - description { return "hello"; }
cy> - (double) f:(int)v  { return v * 0.5; }
cy> @end
cy# o = [new NSObject init]
#"hello"
cy# [o f:3]
1.5
```
* block：`block = ^ int (int value) { return value + 5; }`

### 其他

* choose(Class)：所有 `Class` 的实例。通过寻找 heap 上内存蓝图一致的块来确定是否是实例
* 和 Mobile Substrate 一起使用：@import com.saurik.substrate.MS
* C 交互：使用 dlopen 和 dlsym 加载符号：
```c
dlsym(RTLD_DEFAULT, "swift_demangleClassSimple")
```

## [dumpdecrypted](https://github.com/stefanesser/dumpdecrypted)
