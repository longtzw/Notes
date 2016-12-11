#让自己习惯 C++

##条款01：视 C++ 为一个语言联邦

C++ 的四个主要组成部分：

- C语言
- Object-Oriented C++
- Template C++
- STL

##条款02：尽量以 const，enum，inline 替换 #define

当我们以常量替换 #define，有两种特殊情况值得说说：

第一是定义常量指针。由于常量定义式通常被放在头文件中，因此有必要将指针声明为 const。

`const char* const authorName = "tan zhiwei";`

第二个值得注意的是类的专属常量，为了确保此变量至多只有一份实体，你必须让它成为一个 static 成员。

```c++
class GamePlayer {
private:
    static const int NumTurns = 5; // 常量声明式
    int  scores[NumTurns];         // 使用该常量
}
```

若类的专属常量为整数类型（例如： int, bool），则只要不取常量的地址，可以不提供定义式。其他类型的常量，必须提供
定义式。

##条款03：尽可能使用 const

如果关键字 const 出现在星号左边，表示被指物是常量；如果出现在星号右边，表示指针自身是常量。
后者表示这个指针不得指向不同的东西，但它所指向的东西的指是可以变的。

**const 成员函数**

将 const 实施于成员函数的目的，是为了确认该成员函数可以用作 const 对象身上。

两个成员函数如果只是常量性不同，可以被重载。

有两个流行的概念：bitwise constness（又称 physical constness）和 logical constness。前者认为成员函数只有在不更
改对象之任何成员变量（static 除外）时才可以说是 const。后者认为，一个 const成员函数可以修改它所处理的对象的某些 bits，
但是只有在客户端侦测不出的情况下。

关键字 mutable 可以释放掉 non-static 成员变量的 bitwise constness 约束。

在 const 和 non-const 成员函数中避免重复，可以运用 const 成员函数实现其 non-const 兄弟，但是反之则是不行的。因
为，const 成员函数承诺绝不改变其对象的逻辑状态，non-const 成员函数却没有这般承诺。

- 编译器强制实施 bitwise constness， 但你编写程序时应该使用“概念上的常量性”，mutable 可以帮助你。

##条款04：确定对象被使用前已先被初始化

C++规定，对象的成员变量的初始化动作发生在进入构造函数本体之前。

成员初始化列会调用成员的 copy 构造函数或 default 构造函数。如果成员变量是const 或 reference，它们就一定需要初
值，不能被赋值。

若 class 拥有多个构造函数，多份成员初始化列的存在就会导致重复工作。合理地在初值列中遗漏那些“赋值变现像初始化一
样好”的成员变量，改用它们的赋值操作，并将那些赋值操作移往某个函数，供所有构造函数调用。

不同编译单元内定义之 non-local static 对象的初始化次序无明确定义。改用 Singleton 模式是个不错的解决办法。
