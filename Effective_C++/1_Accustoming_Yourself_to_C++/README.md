# Item 1: C++ 是一个语言联邦

In the beginning, C++ was just C with some object-oriented features
tacked on. Even C++’s original name, “C with Classes,” reflected this
simple heritage.

As the language matured, it grew bolder and more adventurous,
adopting ideas, features, and programming strategies different from
those of C with Classes.

## programming paradigm

Today’s C++ is a **multiparadigm programming language**, one supporting
a combination of **procedural**, **object-oriented**, **functional**, **generic**,
and **metaprogramming** features.

## sublanguage

The easiest way is to view C++ not as a single language but as a federation
of related languages. Within a particular sublanguage, the rules
tend to be simple, straightforward, and easy to remember. When you
move from one sublanguage to another, however, the rules may change.

-  **C**. Way down deep, C++ is still based on C. Blocks, statements, the
    preprocessor, built-in data types, arrays, pointers, etc., all come
    from C. In many cases, C++ offers approaches to problems that
    are superior to their C counterparts (e.g., see Items 2 (alternatives
    to the preprocessor) and 13 (using objects to manage resources)),
    but when you find yourself working with the C part of C++, the
    rules for effective programming reflect C’s more limited scope: no
    templates, no exceptions, no overloading, etc.
- **Object-Oriented C++**. This part of C++ is what C with Classes was
  all about: classes (including constructors and destructors), encapsulation,
  inheritance, polymorphism, virtual functions (dynamic
  binding), etc. This is the part of C++ to which the classic rules for
  object-oriented design most directly apply.
- **Template C++**. This is the generic programming part of C++, the
  one that most programmers have the least experience with. Template
  considerations pervade C++, and it’s not uncommon for rules
  of good programming to include special template-only clauses
  (e.g., see Item 46 on facilitating type conversions in calls to template
  functions). In fact, templates are so powerful, they give rise
  to a completely new programming paradigm, template metaprogramming
  (TMP). Item 48 provides an overview of TMP, but unless
  you’re a hard-core template junkie, you need not worry about it.
  The rules for TMP rarely interact with mainstream C++ programming.
- **The STL**. The STL is a template library, of course, but it’s a very
  special template library. Its conventions regarding containers, iterators,
  algorithms, and function objects mesh beautifully, but templates
  and libraries can be built around other ideas, too. The STL
  has particular ways of doing things, and when you’re working with
  the STL, you need to be sure to follow its conventions.

Keep these four sublanguages in mind, and don’t be surprised when
you encounter situations where effective programming requires that
you change strategy when you switch from one sublanguage to
another. For example, pass-by-value is generally more efficient than
pass-by-reference for built-in (i.e., C-like) types, but when you move
from the C part of C++ to Object-Oriented C++, the existence of userdefined
constructors and destructors means that pass-by-referenceto-
const is usually better. This is especially the case when working in
Template C++, because there, you don’t even know the type of object
you’re dealing with. When you cross into the STL, however, you know
that iterators and function objects are modeled on pointers in C, so for
iterators and function objects in the STL, the old C pass-by-value rule
applies again.

## summary

* Rules for effective C++ programming vary, depending on the part of
  C++ you are using.

# Item 2: 尽量用 const, enum, inline 替换 define 预处理指令

预处理指令 `#define` 并非语言本身的一部分，编译器在开始处理源码前它们就已经被预处理器移除。

## constants

因此由 `#define` 定义的符号并不会进入编译期间的符号表（symbol table），即使编译器报错也无法显示预处理指令定义的相应符号。为此可以使用 `const` 修饰符来定义常量：

```c++
// 预处理阶段会将所有 PI 替换为字面常量 3.14，编译期间只能看到 3.14
#define PI 3.14

// 定义常量，该常量符号会在编译期间写入符号表
const double PiValue = 3.14;
```

## static class constants

为了将常量的作用域（scope）限制在类中，可以将该常量声明为类成员，并且为了保证该类仅含有该常量的一个实体，需要将其声明为静态成员，这就是静态类常量，示例如下：

```c++
class A {
  private:
    static const int N = 5;  	// 声明，一般位于头文件中
    int scores[N];				// 使用该类常量，就是一个类作用域的普通常量而已
  public:
    A();
};
```

如果想要对类常量取地址，则还需要在实现文件中提供该静态类常量的定义：

```c++
const int A::N;		// 在声明时指定了初始值，定义时无须再提供
```

我们是无法使用 `#define` 指令来定义类常量的，因为预处理指令没有作用域的概念。

此外还可以使用 the enum hack 方法来定义类常量，示例如下：

```c++
class A{
  private:
    enum { N=5 };		// 定义的 enum 作用域在当前类
    int scores[N];
  public:
    A();
};
```

跟使用 `const` 定义的类常量相比，`enum` 类常量是无法进行取地址的，也不会分配额外的内存空间。

## inline vs. macro

在 C 语言时代，人们经常用 `#define` 创建宏（macro）来避免函数调用的开销。不过宏有太多的缺陷了，在 C++ 时代，我们可以通过内联来获得宏带来的效率，同时又具有编译期的类型安全性检查。示例如下：

```c++
template<typename T>
inline T max(const T&x, const T& y) {
    return x>y ? x : y;
}
```

## summary

* 用 `const` 或 `enum` 代替 `#define` 来定义常量
* 用 `inline` 函数来代替 `#define` 定义的宏

# Item 3: 尽可能的使用 const

`const` 用来指定一个语义约束（即对象不能被改动），编译器会强制实施这项约束。也即允许你告诉编译器和其它程序员某个对象的值不能改动，编译器会确保该约束不会被违反。

## const and pointer/iterator

const 既可以约束指针不可改动（const pointer），也可以约束指针指向的对象不可改动（const data）。示例如下：

```c++
char s[] = "hello";
const char* p = s;	// non-const pointer, const data
char* const q = s;  // const pointer, non-const data
const char* const ps = s;	// const pointer, const data
```

这里的关键在于 `const` 修饰符出现在星号前还是后，出现在星号前则表示指针指向的对象不可变，出现在星号后则表示指针不可变。

`const` 出现在星号前，又有两种写法，它们是等价的，只是书写风格不同而已：

```c++
const char* p = s;
char const* p = s;
```

STL 中的迭代器构建于指针的概念之上，因此迭代器也有自身不可变和指向对象不可变的区分，示例如下：

```c++
std::vector<int> v;

std::vector<int>::iterator iter = v.begin();		// non-const iterator, non-const data
std::vector<int>::const_iterator iter = v.begin();	// non-const iterator, const data
const std::vector<int>::iterator iter = v.begin();	// const iterator, non-const data
```

## const and function

函数的返回值设为常量，可以降低由于客户错误而造成的意外，如下示例：

```c++
class Rational {};
const Rational::operator*(const Rational& lhs, const Rational& rhs) {}
```

这里我们设置 `operator*` 返回常量，可以避免客户试图对乘法返回结果进行改动，如：

```c++
Rational a, b, c;
(a*b) = c;	// 不被允许
```

## const member function

这里指的并不是成员函数的返回值为常量，而是用 `const` 修饰成员函数，其语义是：该成员函数不可更改当前对象内任何的 not-static 成员变量。

此外两个成员函数如果仅仅只是常量性不同，也是可以被重载的，如下示例：

```c++
class TextBlock {
  private:
    std::string text;
  public:
    TextBlock(const char* str);
    char& operator[](std::size_t i) { return text[i]; }
    char& operator[](std::size_t i) const {  // 常量成员函数
        return text[i];
    }
};

TextBlock tb("hello");
const TextBlock ctb("hello");

tb[0];	// 调用 non-const operator[]
ctb[0];	// 调用 const operator[]
```

在上例中 const operator[] 函数的返回值是一个引用，客户可以用该引用修改对象，显然这是不符合 const 成员函数的意图的，为此需要将其返回值也改为常量：

```c++
const char& TextBlock::operator[](std::size_t i) const {
    return p[i];
}
```

假定重载的非常量和常量成员函数在返回数据之前要进行一系列边界检验、数据完整性检查等处理工作（二者的实质工作内容是相同的，仅仅区别在于常量性），如果在两个成员函数中重复同样的逻辑，不仅冗余，也不利于以后的维护，当然我们可以将共同的逻辑抽取出来放入一个 private 成员函数中，再调用它。不过这里介绍一种更优雅的方法：将完整的逻辑放入常量成员函数中，然后非常量成员函数通过调用并类型转换来实现，示例如下

```c++
class TextBlock {
  private:
    std::string text;
  public:
    char& operator[](std::size_t i) {
        return const_cast<char &>(
            static_cast<const TextBlock&>(*this)[i]
        );
    }
    const char& operator[](std::size_t i) const {
    	// bounds checking
        // verify data
        return text[i]; 
    }
};
```

直到现在我们对常量性的定义其实又叫做 physical constness 或 bitwise constness（即常量成员函数内不能够改动对象的任何一个 bit），此外还有 logical constness：在不被客户感知的情况下，常量成员函数可以修改对象的某些成员。假设 `CTextBlock` 支持对文本块长度的缓存，以供客户频繁的访问块长度，也就说在每次查询文本块长度时，需要更新长度信息（更改某些成员变量），同时长度查询操作对于客户而言应该具有常量性。我们需要借助 `mutable` 来释放掉 non-static 成员变量的 bitwise constness 约束，示例如下：

```c++
class CTextBlock {
  public:
    std::size_t length() const;	// 常量成员函数
  private:
    char *pText;
    mutable std::size_t textLength;	// 声明可以被常量成员函数改动
    mutable bool lengthIsValid;		// 声明可以被常量成员函数改动
};

std::size_t CTextBlock::length() const {
    if (!lengthIsValid) {
        // 修改 mutable 成员变量
        lengthIsValid = true;
        textLength = std::strlen(pText);
    }
    return textLength;
}
```

# Item 4: 确定对象被使用之前已经完成了初始化

对象在被定义时不一定会自动进行初始化：

> In general, if you’re in the C part of C++ (see Item 1) and initialization
> would probably incur a runtime cost, it’s not guaranteed to take
> place. If you cross into the non-C parts of C++, things sometimes
> change. This explains why an array (from the C part of C++) isn’t necessarily
> guaranteed to have its contents initialized, but a vector (from
> the STL part of C++) is.

读取未初始化的对象是未定义的行为（undefined behavior），在某系统上可能会导致宕机，或者读取一些随机内容。我们要谨记：永远要在使用对象之前先将它初始化。

## initialization and assignment

对于内置类型，我们必须手动完成初始化；对于自定义类型，可以通过构造函数（constructor）来完成初始化，不过这里有点要注意：不要将初始化跟赋值相混淆。

```c++
class PhoneNumber {};
class ABEntry {
  public:
    ABEntry(const std::string& name, 
            const std::string& address, 
            const std::list<PhoneNumber>& phones);
  private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int count;
};

ABEntry::ABEntry(const std::string& name, 
        		 const std::string& address, 
                 const std::list<PhoneNumber>& phones) {
    theName = name;
    theAddress = address;
    thePhones = phones;
    count = 0;
}
```

以上构造函数在函数体内进行的操作并非对成员变量的初始化，而是赋值操作。这是因为 C++ 规定，成员变量的初始化在进入构造函数体内之前已经完成。因此上例的真实行为是，成员变量 `theName`, `theAddress`, `thePhones` 在进入构造函数体之前，已经调用 default 构造函数完成了初始化，然后在构造函数体内再次调用 copy assignment 操作符完成赋值操作。显然对成员变量进行一次初始化，再进行一次赋值是冗余低效，甚至可能是错误的。

## member initialization list

为了精确的初始化成员变量，不应该在构造函数体内进行赋值，而是应该使用成员初始化列表（member initialization list）语法，如下所示：

```c++
ABEntry::ABEntry(const std::string& name, 
        		 const std::string& address, 
                 const std::list<PhoneNumber>& phones):
	theName(name),
	theAddress(address),
	thePhones(phones),
	count(0)
	{ }
```

本例中成员变量 `theName`, `theAddress`, `thePhones` 在初始化列表中调用了 copy 构造函数来完成了初始化。

我们还可以实现一个无参数的构造函数：

```c++
ABEntry::ABEntry(const std::string& name, 
        		 const std::string& address, 
                 const std::list<PhoneNumber>& phones):
	theName(),
	theAddress(),
	thePhones(),
	count(0)
	{ }
```

本例中成员变量 `theName`, `theAddress`, `thePhones` 在初始化列表中调用了 default 构造函数来完成了初始化。

一般情况，不论是内置类型还是自定义类型的成员变量，都建议通过成员初始化列表来进行精确的初始化。如果不同的构造函数都有许多相同的成员初始化动作，可以考虑将这些成员变量从列表中移除，用赋值操作来完成“初始化”，并将赋值操作放入某个 private 函数中，然后由这些构造函数调用该函数。如：需要从配置文件中加载配置项。

## the order of member initialization

成员初始化次序：基类（base class）成员早于派生类（derived class），且一个类中成员变量的初始化次序跟成员变量在类中声明的顺序相对应，跟成员初始化列表的次序无关。为了可读性的考虑，建议成员初始化列表的次序跟成员的声明次序保持一致。

## non-local static object initialization in different translation units.

C++ 规定，定义于不同编译单元内的 non-local static 对象之间的初始化次序是不确定的。

static 对象的声明周期从程序运行开始，到程序结束时终止。static 对象根据作用于可以分为 local static 和 non-local static，我们将定义在函数内部的 static 对象称为 local 的，其余的称为 non-local 的（如：定义于全局作用域的，定义于 namespace 作用域，定义于文件作用域的）。

编译单元（translation unit）是指生成单一目标文件（object file）所对应的源码。

例：有一个库定义了文件系统对象

```c++
class FileSystem {
  public:
    std::size_t numDisks() const;
};

extern FileSystem tfs;		// 这里是声明，其定义可能是位于 namespace 作用域的静态对象，以表示全局唯一的文件系统对象
```

客户使用该对象来处理文件系统目录

```c++
class Directory {
  public:
    Directory() {
        // 如何保证此时 tfs 已经完成了初始化
        std::size_t disks = tfs.numDisks();
    }
};

Directory tempDir();
```

这里我们无法保证在创建 `tempDir` 对象时，`tfs` 已经完成了初始化。

我们可以使用单例模式（singleton）来解决这个问题，如下：

```c++
class FileSystem {
  public:
    std::size_t numDisks() const;
};

FileSystem& tfs() {
    static FileSystem fs;
    return fs;
}
```

要使用文件系统实例，就调用 `tfs()` 函数来获得，这样保证了使用它之前，初始化一定已经完成。

## summary

- 内置类型要手动初始化
- 构造函数最好用成员初始化列表进行初始化，尽量避免在构造函数体内进行赋值操作
- 出于可读性的考虑，成员初始化列表的顺序最好跟类中声明的顺序相一致
- 用单例模型和 local static 来避免 non-local static 对象初始化顺序的不确定性









