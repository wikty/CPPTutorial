# Terminology

## Declaration

声明（declaration）就是告诉编译器某个东西的**名称**和**类型**，但略去细节，如下示例：

```c++
extern int x;		// 对象（object）声明
int f(int, int);		// 函数（function）声明：指定函数的签名
class A;			// 类（class）声明
template<typename T>
class Node;			// 模板（template）声明
```

## Definition

定义（definition）为编译器提供声明所遗漏的细节，如下示例：

```c++
int x;					// 对象（object）定义
int f(int a, int b) {	// 函数（定义）：定义函数体
    return a + b;
}
class A {				// 类（class）定义：定义类的成员
  public:
    A();
    ~A();
    // ...
};
template<typename T>	// 模板（template）定义：定义模板的成员
class Node {
  public:
    Node();
    ~Node();
    // ...
};
```

## Initialization

初始化（initialization）就是给对象赋予初始值的过程，初始化一般由构造函数（constructor）来完成。

### default constructor

default 构造函数就是一个无须传入参数就可以被调用的构造函数。可能该函数本来就不接受参数或者可能是参数都有默认值。

### copy constructor vs. copy assignment operator

copy 构造函数用同类型的对象来初始化当前对象。copy 赋值操作运算符则是将同类型对象的内容拷贝至当前对象，如下所示：

```c++
class A {
  public:
  	A();						// default constructor
    A(const A& rhs);			// copy constructor
    A& operator=(const A& rhs); // copy assignment operator
};

A a1;		// 调用 default constructor
A a2(a1);	// 调用 copy constructor
A a3 = a1;  // 调用 copy constructor
a3 = a2;	// 调用 copy assignment operator
```

调用 copy constructor 和 copy assignment operator 的关键区别在于：是否有新对象被定义。

### explicit specifier for constructor

默认情况下，当一个对象需要被构造时会执行隐式类型转换（implicit type conversion），如下：

```c++
class A {
  public:
    A(int x);
};

void f(A a);

f(20);  // 执行了隐式类型转换 A(20)
```

如果不希望执行隐式类型转换的话，需要声明构造函数时指定 `explicit`，如下：

```c++
class A {
  public:
    explicit A(int x);
};

class B {
  public:
    explict B(int x = 0, int y = 0);
};
```

将构造函数声明为 `explicit` 可以禁止编译器执行非预期的隐式类型转换，使错误尽早的在编译器期间暴露出来，推荐这种做法。

## Undefined behavior

C++ 标准对有些行为没有明确的定义，如：读取空指针。

