# C and CPP How To Return an Anonymous Structure
C与C++，如何定义一个函数返回匿名结构体

<br />

从C++11开始，C++语言核心中加入了数十种新特性，其中就有类型自动推导语法系统。在该系统中，将原本的`auto`关键字用作了对所定义的对象进行类型推导的声明；还有`decltype`关键字用于获取指定对象的类型。通过结合这一语法特性，我们可以定义一个函数，使得它能返回一个匿名结构体对象，并且也能很容易地让调用者去使用。

那么有些朋友会问，为何要返回一个匿名结构体对象呢？因为匿名结构体能减少全局名字空间的污染，尽管在C++中我们可以自己定义名字空间，但显然使用匿名结构体会省事得多；同时，C++没有原生的元组类型，尽管可以通过标准库使用 **std::tuple**，但是它会更耗费存储空间，同时也影响运行时效率，不如结构体效率高。因此，如果我们要让一个函数返回多个值，那么匿名结构体将是非常好的对原生元组的补充。

关于C++中如何定义一个返回类型为匿名结构体对象的代码如下所示：

```cpp
#include <iostream>
using namespace std;

// 我们可以直接定义一个返回匿名结构体的函数
static struct { int a; int b; } Foo(int a, int b)
{
    // 利用decltype来获取函数的返回类型
    decltype(Foo(0, 0)) const s = { a, b };
    return s;
}

// 当然，我们可能更多的是先声明函数
static struct { int a; int b; } FooStruct(int a, int b);

// 如果之前该函数已经被声明了，
// 那么我们可以用类型推导的方式来做函数定义，这样更直观些
static auto FooStruct(int a, int b) -> decltype(FooStruct(0, 0))
{
    decltype(FooStruct(0, 0)) const s = { a, b };
    return s;
}

int main(void)
{
    auto s1 = Foo(1, 2);
    cout << "a = " << s1.a << ", b = " << s1.b << endl;

    auto s2 = FooStruct(10, 20);
    cout << "a = " << s2.a << ", b = " << s2.b << endl;

    // 这里直接使用了结构体默认支持的std::initializer_list语法特性
    auto s3 = decltype(s2){ 5, 6 };
    cout << "The value is: " << s3.a + s3.b << endl;
}
```

对于上述代码，各位可以在Visual Studio 2017或更高版本。当前Clang、GCC还没支持上述这种表达形式，因为在当前稳定的C++版本中，C++标准尚未将此特性加入，而在2017年02月03日，有人已经提出了这个决议，详细请见此决议文档：《[Implicit Return Type and Allowing Anonymous Types as Return Values](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/p0536r0.html)》，但估计不太会实现了。因为在C++14标准中引入了通过 `auto` 关键字声明函数，然后缺省为它指定类型，该函数将会通过返回值自动推导出类型。不过这个机制的一大缺陷就是，在调用函数时必须要先知道这个函数的定义，否则编译器是无法知道该函数的确切返回类型的，因此这个机制一般用于当前源文件的部分功能实现，要作为外部共享函数的话，要么通过静态函数方式，要么就用内联形式来定义了。

下面给出例子：

```cpp
#include <iostream>
using namespace std;

static auto Foo(int a, int b)
{
    struct
    {
        int x, y;
    } ret{a, b};
    
    return ret;
}

int main(void)
{
    auto const s = Foo(1, 2);
    cout << "The value is: " << s.x + s.y << endl;
}
```

<br />

下面谈谈C语言中如何实现。由于当前纯正的C18标准是不支持类型推导以及获取对象类型的语法的，因此我们直接用C11/C18标准是行不通的。但是GNU语法扩展很早就引入了 `typeof` 关键字，`typeof` 的操作数不仅可以为对象表达式，而且还可以是类型，所以灵活度上比C++的 `decltype` 更高。从GCC 4.9以及Clang 3.8起，这两大开源主流编译器就引入了 `__auto_type` 关键字，其作用就跟C++11标准中的 `auto` 一样，作为类型自动推导的声明符。这么一来就能在使用返回类型为匿名结构体的函数调用上更为简洁了。

下面看一下基于gnu11标准的C代码：

```c
#include <stdio.h>

#ifndef let
#define let     __auto_type
#endif // !let


static struct { int a; int b; } FooStruct(int a, int b)
{
    typeof(FooStruct(0, 0)) s = { a, b };
    return s;
}

int main(void)
{
    let const s = FooStruct(10, 20);
    printf("a = %d, b = %d\n", s.a, s.b);
    
    // 这里使用了匿名结构体字面量和指定成员的初始化器这两种语法特性
    let const s2 = (typeof(s)){ .a = 1, .b = 2 };
    printf("The value is: %d\n", s2.a + s2.b);
}
```

以上代码需要GCC 4.9或更高，Clang 3.8或更高，VS 2017或更高版本并使用Clang编译器模板进行编译运行。


