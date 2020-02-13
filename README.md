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
}
```

对于上述代码，各位可以在Visual Studio 2017或更高版本，GCC 4.9或更高版本，Clang 3.8或更高版本中编译运行。

<br />

下面谈谈C语言中如何实现。
