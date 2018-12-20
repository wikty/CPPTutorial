> Software designs — approaches to getting the software to do what you
> want it to do — typically begin as fairly general ideas, but they eventually
> become detailed enough to allow for the development of specific
> interfaces. These interfaces must then be translated into C++ declarations.
> In this chapter, we attack the problem of designing and declaring
> good C++ interfaces.

# Item 18: 让接口容易被正确使用，不易被误用

客户通过函数、类、模板等各种接口同你的代码进行交互。理想情况下，如果客户企图使用接口却没能获得他所期望的行为，代码不应该通过编译。

下面将通过若干例子说明，在设计接口时采取哪些机制可以预防客户误用该接口：

## 类型安全

通过限制接口参数的类型来预防接口被误用，如下：

```c++
class Date {
  public:
    Date(int month, int day, int year);
};
```

该构造函数就极容易被误用，客户不一定会记得清楚这三个参数的顺序，我们可以通过引入新类型来预防误用：

```c++
class Day {
  public:
    explicit Day(int);
};

class Month {
  public:
    explicit Month(int);
};

class Year {
  public:
    explicit Year(int);
};

class Date {
  public:
    Date(Month m, Day d, Year y);
};

Data d(Month(1), Day(1), Year(2009));
```

进一步我们还可以限制参数取值：

```c++
class Month {
  public:
    // 通过静态方法创建对象
    static Month Jan() { return Month(1); }
    static Month Feb() { return Month(2); }
    // ...
    static Month Dec() { return Month(12); }
  private:
    explicit Month(int m);  // 禁止外部访问构造函数
};

Data d(Month::Jan(), Day(1), Year(2009));
```

返回值可以施加 `const` 限制，防止被修改：

```c++
class Integer {
  public:
    const Integer operator+(const Integer& lhs, const Integer& rhs);
};

Integer i, j;

(i+j) = 2; // error!
```

## 接口一致性

自定义类型的接口应该尽量同内置类型的相一致，满足用户使用习惯，避免被误用。或者更一般地，库开发者应该保证接口风格的一致性，比如 STL 的接口就比较一致：许多容器对象都支持 `size()` 方法， 常量迭代器 `const_iterator` 等。

## 消除用户的资源管理责任

任何接口如果期望用户记得必须做某些事，该接口可能就存在问题，应该尽量降低用户责任。

错误示范：

```c++
// 工厂函数创建投资品
Investment* createInvestment();
```

寄希望于客户记得删除由工厂函数创建的对象或将该对象放入智能指针中，使得客户负担加重，客户可能忘记删除该对象或者多次删除。接口开发者可以利用垃圾回收机制来减轻客户负担，将接口设计如下：

```c++
std::shared_ptr<Investment> createInvestment();
```

倘若资源的释放不是简单的 `delete`，而是需要通过函数 `getRidOfInvestment()` 来进行，同样不应该将资源释放的负担转嫁给客户，可以这样改进接口：

```c++
std::shared_ptr<Investment> createInvestment() {
    std::shared_ptr<Investment> p(static_cast<Investment* >(0), getRidOfInvestment);
    
    // point p to a Investment object
    
    return p;
}
```

# Item 19: 设计类就是在设计类型系统

In C++, as in other object-oriented programming languages, defining a
new class defines a new type. Much of your time as a C++ developer
will thus be spent augmenting your type system. This means you’re
not just a class designer, you’re a type designer. Overloading functions
and operators, controlling memory allocation and deallocation, defining
object initialization and finalization — it’s all in your hands. You
should therefore approach class design with the same care that language
designers lavish on the design of the language’s built-in types.
Designing good classes is challenging because designing good types is
challenging. Good types have a natural syntax, intuitive semantics,
and one or more efficient implementations. In C++, a poorly planned
class definition can make it impossible to achieve any of these goals.
Even the performance characteristics of a class’s member functions
may be affected by how they are declared.
How, then, do you design effective classes? First, you must understand
the issues you face. Virtually every class requires that you confront
the following questions, the answers to which often lead to
constraints on your design:

- How should objects of your new type be created and destroyed?
- How should object initialization differ from object assignment?
- What does it mean for objects of your new type to be passed by value?
- What are the restrictions on legal values for your new type?
- Does your new type fit into an inheritance graph?
- What kind of type conversions are allowed for your new type?
- What operators and functions make sense for the new type?
- What standard functions should be disallowed?
- Who should have access to the members of your new type?
- What is the “undeclared interface” of your new type?
- How general is your new type?
- Is a new type really what you need?

# Item 20: 用 pass-by-reference-to-const 替换 pass-by-value

C++ 中，函数的参数传递和返回值都是以 pass-by-value 方式进行的，也即意味着在函数调用过程中会有对象复制成本。因此我们建议，尽量使用引用来传递参数。

## 降低参数传递成本

```c++
class Person {
  private:
    std::string name;
    std::string address;
};

class Student: public Person {
  private:
    std::string schoolName;
    std::string schoolAddress;
};

Student f(Student s);

Student stu;
Student stu1;
stu1 = f(std);
```

此例中函数调用时调用了 copy 构造函数来创建形参对象以及内部字符串对象，并在函数结束调用后销毁这些对象；函数返回时，调用了 copy 复制函数来保存返回数据。可见通过 pass-by-value 的成本较高。

解决之道是尽量用 pass-by-reference-to-const 来替代 pass-by-value：

```c++
Student f(const Student& s);
```

经过以上改进，调用函数时形参仅仅是实参对象的常量引用，不会存在构造和析构的成本（不过这里函数返回值依然需要 copy 复制）。

## 避免对象分割问题

如果某个接口接受基类参数，而传入的实参却是派生类时，会发生什么？当传入派生类对象时，将会构造形参对象且该对象属于基类而非派生类。

```c++
// 普通窗口
class Window {
  public:
    std::string name() const;
    // 专属的 display
    virtual void display() const;
};

// 滑动窗口
class WindowWithScrollBar: public Window {
  public:
    // 专属的 display
    virtual void display() const;
};

void printNameAndDisplay(Window w) {
    std::cout << w.name();
    w.display();
}

WindowWithScrollBar sw;
printNameAndDisplay(sw);  // 实参传入派生类对象，但构造的形参对象却是基类的
```

要解决该问题，只需要让接口按照 pass-by-reference-to-const 传参即可：

```c++
void printNameAndDisplay(const Window& w) {
    std::cout << w.name();
    w.display();
}
```

## 哪些类型不适宜传引用

C++ 编译器底层一般通过指针来实现引用，因此对于内置类型（如：int, float）来说，反而传递引用的成本会更高。此外 STL 中的迭代器和函数对象也最好使用 pass-by-value 方式。

# Item 21: 函数返回引用

上一节中我们提到函数返回值也会伴随着复制行为，那么我们是否可以通过返回引用来避免这个复制成本呢？许多情况下，返回一个引用是比较危险的，见下例：

```c++
class Rational {
  public:
    const Rational& operator*(const Rational& lhs, const Rational& rhs) {
        Rational result(lhs.n*rhs.n, lhs.d*rhs.d);
        return result;
    }
};

Rational a(1, 2), b(3, 4);
Rational c = a * b;
```

该成员函数返回一个引用指向某个局部对象，可是此局部对象在成员函数返回时已经被销毁了，那么返回值所引用的东西到底是什么！

也许你觉得既然局部对象会被销毁，那动态创建一个对象吧：

```c++
class Rational {
  public:
    const Rational& operator*(const Rational& lhs, const Rational& rhs) {
        Rational* p = new Rational(lhs.n*rhs.n, lhs.d*rhs.d);
        return *p;
    }
};
```

虽然在函数返回时，动态创建的对象不会被销毁，但这个对象应该何时销毁？由谁来销毁？忘记销毁怎么办？此外像下面这样的链式乘法一定会泄漏内存：`d=a*b*c`。

既然动态对象不可行，那么静态对象呢？在函数内创建一个静态对象，它可是在程序生命周期里一直存在的：

```c++
class Rational {
  public:
    const Rational& operator*(const Rational& lhs, const Rational& rhs) {
        static Rational result;
        result.n = lhs.n*rhs.n;
        result.d = lhs.d*rhs.d;
        return result;
    }
};
```

首先静态对象存在多线程安全问题，此外它使得 `(a*b) == (c*d)` 总是为真，不管取值是怎样的，总是返回同一个静态对象的引用。

显然我们不该为了避免返回值的复制成本，而想这么多奇技淫巧，还是老老实实的接受它比较好：

```c++
class Rational {
  public:
    const Rational operator*(const Rational& lhs, const Rational& rhs) {
        return Rational (lhs.n*rhs.n, lhs.d*rhs.d);
    }
};
```

# Item 22: 成员变量声明为私有

成员变量既不应该是 public，也不应该是 protected，而应该声明为 private。成员变量声明为私有，有如下好处：

## 语法一致性

从语法一致性上来看，公有成员变量的存在，使得客户需要花成本去辨识某个接口是成员函数还是成员变量。

## 精细化控制

从精细化控制角度来看，将成员变量设为公有的话，客户对该成员变量将拥有完全的读写权限。但私有成员变量却可以通过暴露接口来实现更加精细化的控制，如下：

```c++
class A {
  private:
    int noAccess;
    int readOnly;
    int readWrite;
    int writeOnly;
  public:
    int getReadOnly() const { return readOnly; }
    int getReadWrite() const { return readWrite; }
    void setReadWrite(int value) { readWrite = value; }
    void setWriteOnly(int value) { writeOnly = value; }
};
```

## 封装隔离变化

私有化成员变量，然后通过函数接口的形式进行交互，可以极大程度上的隔离类内部的变化，使得客户不会受到影响。

将成员变量隐藏在函数接口之后，可以为所有可能的实现提供弹性。如：控制读写，检验值的有效性，多线程中控制同步等。

成员变量被封装后，可以降低对系统的破坏。对于公有成员变量，如果改动的将导致，客户依赖的代码重写，但对于私有成员来将，却可以将破坏性尽量降低。

# Item 23: 用非成员且非友元函数来替换成员函数

有时可能需要为类提供一些新的功能，但并不一定就应该将其实现为类的成员函数。现在我们有一个浏览器的类：

```c++
class WebBrowser {
  public:
    void clearCache();
    void clearHistory();
    void clearCookies();
};
```

现在我们希望可以有一个接口可以一次性清除缓存、历史和 cookie，而不是分别调用以上三个接口，那么我们应该为类添加这样一个成员函数吗：

```c++
class WebBrowser {
  public:
    void clearCache();
    void clearHistory();
    void clearCookies();
    
    void clearEverything() {
        clearCache();
        clearHistory();
        clearCookies()
    }
};
```

还是提供一个非成员且非友元的函数呢：

```c++
void clearEverything(WebBrowser& wb) {
    wb.clearCache();
    wb.clearHistory();
    wb.clearCookies()
}
```

我们更加推荐后者，原因有两点：

- 封装性更好
- 更加易于扩展

## 封装性

面向对象守则要求数据应该尽可能的被封装，良好的封装性意味着，我们将有更大的弹性对未来的需求作出变化。

在一个类的内部，我们可以认为越少接口可以看到数据，则该类的封装性越好。因此不论是通过添加成员函数，还是友元函数，来为类增加新的功能，显然都会弱化类的封装性。而通过非成员且非友元函数添加新功能，则很好的维护了类的封装性。

值得注意的是，这里非成员且非友元函数，不一定是全局函数，它也有可能是其它类中的成员，比如：为 `WebBrowserUtils` 类添加 `static void clearEverything();` 方法。

## 扩展性

上述定义的非成员且非友元函数 `clearEverything()` 以及类 `WebBrowser` 一般我们会将它们都放在名称空间中：

```c++
namespace WebBrowserStuff {
    class WebBrowser { 
    	// ...
    };
    
    void clearEverythin(WebBrowser& wb);
}
```

C++ 支持同一个名称空间定义在多个源文件中，这样使得我们可以将不同功能的接口定义在不同源文件中但同时却隶属于同一名称空间，这样可以降低编译依赖关系的复杂度：

```c++
// "webbrowser.h"
// core components for web browser
namespace WebBrowserStuff {
	class WebBrowser {};
}

// "webbrowser_bookmarks.h"
namespace WebBrowserStuff {
	// non-member for bookmarks
}

// "webbrowser_cookies.h"
namespace WebBrowserStuff {
	// non-member for cookies
}
```

并且当客户想要增添自己的功能函数时，只需要继续向该名称空间添加非成员且非友元函数即可，可见这种方式为系统提供了极大的可扩展性。

# Item 24: 用非成员函数支持所有参数的类型转换

在构建数值类的类型时，我们往往希望类型能够实现隐式类型转换。比如我们要构建一个有理数类，那么我们希望整数在同有理数运算时，可以实现自动（隐式）类型转换。

```c++
class Rational {
  public:
    // non-explicit 可以支持整数到有理数的自动类型转换
    Rational(int numberator = 0, int denominator = 1);
    int n() const;
    int d() const;
};
```

如果我们想要实现有理数乘法呢：

```c++
class Rational {
  public:
    const Rational operator*(const Rational& rhs) const;
};

Rational a(1, 2);
Rational b(3, 4);
Rational c;

c = a * b; // ok
c = a * 2; // ok: int to Rational implicit
c = 2 * a; // error: not supprot operator*(int, Rational)
```

对于最后一种情况的整数和有理数混合运算，将以报错收场，由于我们并没有定义支持此运算的函数（成员函数不支持，全局函数也不支持）。要想支持此类运算，我们可以定义一个非成员函数：

```c++
const Rational operator*(const Rational& lhs, const Rational& rhs) {
    return Rational(
    	lhs.n() * rhs.n(),
        lhs.d() * rhs.d()
    );
}
```

# Item 25: 安全的 swap 函数











