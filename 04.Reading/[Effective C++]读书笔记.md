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

#构造/析构/赋值运算

##条款05：了解C++默默编写并调用哪些函数

编译器产出的析构函数是个 non-virtual。

编译器拒绝为以下情况生成对应的 copy 构造函数和赋值构造函数：

- 内含 reference 成员的 class。
- 内含 const 成员的 class。
- base class 将 copy 或 赋值构造声明为 private 的 derived class。

##条款06：若不想使用编译器自动生成的函数，就该明确拒绝

将成员函数声明为 private 而且故意不实现它们。

##条款07：为多态基类声明 virtual 析构函数

C++ 明确指出，当 derived class 对象经由一个 base class 指针被删除，而该 base class 带着一个 non-virtual 析构函数，其结果未定义。

不要企图继承一个标准容器或任何其他“带有 non-virtual 析构函数”的class。

当你希望拥有抽象 class，你可以为它声明一个 pure virtual 析构函数，且必须为其提供一份定义，否则会出现链接错误。

Class 的设计目的如果不是作为 base classes 使用，或不是为了具备多态性，就不该声明 virtual 析构函数。

##条款08：别让异常逃离析构函数

- 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们或结束程序。
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么 class 应该提供一个普通函数（而非在析构函数中）执行该操作。

##条款09：绝不在构造和析构过程中调用 virtual 函数

##条款10：令 operator= 返回一个 reference to \*this

为了实现“连锁赋值”，赋值操作符必须返回一个 reference 指向操作符左侧的实参。

##条款11：在 operator= 中处理“自我赋值”

让 operator= 具备“异常安全性”往往自动获得“自我赋值安全”的回报。一群精心安排的语句就可以导出异常安全的代码。

```c++
Widget& Widget::operator=(const Widget& rhs)
{
     Bitmap *pOrig = pb;
     pb = new Bitmap(*rhs.pb);
     delete pOrig;
     return *this;
}
```

##条款12：复制对象时勿忘其每一个成分

如果你为 class 添加一个成员变量，你必须同时修改 copying 函数。

- Copying 函数应该确保复制“对象内的所有成员变量”及“所有 base class 成分”。
- 不要尝试以某个 copying 函数实现另一个 copying 函数。应该将共同机能放进第三个函数中，并由两个 copying 函数共同调用。
