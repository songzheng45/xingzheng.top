---
title: C# Lambda表达式和表达式树
tags: 
    - C#
    - Lambda
    - 表达式树
category: 技术
---

## Lambda简化匿名类型
### 匿名类型写法：  
``` c#
Func<string,int> returnLength;
returnLength = delegate(string text){return text.Length;};
```

### lambda 显式参数列表语法：  
*(explicitly-typed-parameter-list) => expression*  
``` c#
Func<string,int> returnLength;
returnLength = (string text) => {return text.Length;};
```

<!--more-->

### lambda 隐式参数列表语法：  
*(implicitly-typed-parameter-list) => expression*  
各个参数名之间以逗号分隔，无需类型，由编译器推断参数的类型。  
但是如果任何一个参数是*out* 或 *ref* 的，则必须使用显式类型。
``` c#
Func<string,int> returnLength;
returnLength = （text) =>  text.Length;
```

### 单个参数的简短语法
如果lambda表达式只需要一个参数，那么参数可以是隐式类型，C#3还允许省略圆括号，如下形式：  
*parameter-name => expression*   
最终的lambda表达式形式如下：  
``` c#
Func<string,int> returnLength;
returnLength = text =>  text.Length;
```

当方法的参数是委托类型时，lambda表达式可以作为参数传递，而实际上编译器会生成一个方法，然后用该方法创建一个委托实例作为参数。等同于这里传入一个匿名方法。
``` c#
films.FindAll(film=>film.Year<1960);
```
编译器生成的代码如下：
``` c#
private static bool SomeAutoGeneratedName(Film film) {
   return film.Year < 1960;
}
films.FindAll(new Predicate<Film>(SomeAutoGeneratedName))
```

## 表达式树
.NET 3.5 推出的表达式树提高了一种抽象的方式，将一些代码作为对象树来对待。它类似 CodeDOM，但是它的操作处于一个稍高的级别。  
表达式树的主要应用就是LINQ。    
C# 3 支持从lambda表达式到表达式树的转换，它不借助任何编译器的巧妙处理而融入了.NET Framework。  

### 编程的方式构建表达式树  
It’s worth noting that the leaf expressions are created first in the code。  
值得注意的是，叶子表达式首先在代码中创建。表达式一旦创建则不可改变，因此你可以随意缓存和重用表达式。    
最简单的表达式树例子，2和3相加：
``` c#
using System.Linq.Expressions;

Expression firstArg = Expression.Constant(2);
Expression secondArg = Expression.Constant(3);
Expression add = Expression.Add(firstArg,secondArg);
Console.WriteLine(add);
```
输出：`(2 + 3)`。  

### 把表达式树编译成委托
``` c#
Expression firstArg = Expression.Constant(2);
Expression secondArg = Expression.Constant(3);
Expression add = Expression.Add(firstArg,secondArg);

Func<int> compiled = Expression.Lambda<Func<int>>(add).Compile();
Console.WriteLine(compiled());
```
虽然我们可能永远也不会像这样来使用表达式树，甚至也不会以编程的方式来构造表达式树，但是这些底层的信息能帮我们更好的理解LINQ是如何工运作的。

### 把C# lambda 表达式转换成表达式树
``` c#
Expression<Func<int>> return5 = () => 5;
Func<int> result = return5.Compile();
Console.Write(result());
```
>并不是所有的lambda表达式能被转换为表达式树。  
>例如，不能将带代码块的lambda表达式（即使只有一个return语句）转换为表达式树。


### 表达式树是LINQ的核心
lambda表达式、表达式树和扩展方法的组合结果就是“LINQ”。  
长久以来，我们要么只能获得很好的编译时检查，要么只能告诉其他平台去执行一些代码，通常用文本表示（最明显的例子是SQL）。但是两者不可得兼。  
通过组合lambda表达式，可以提供表达式树的编译时检查，由此从期望的逻辑中抽象出执行模型。在合理的范围内我们可以同时拥有它们的好处。  
在进程外LINQ提供者的核心是，你可以从熟悉的语言（这里指C#）产生一个表达式树，将其结果作为一个中间格式，然后被转换为特定平台的本地语言。例如SQL。
在某些情况下，可能没有简单的本机语言与本地API一样，也可能会根据表达式产生不同的Web服务调用。
