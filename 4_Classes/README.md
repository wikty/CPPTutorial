[TOC]

# Classes

Classes are an expanded concept of *data structures*: like data structures, they can contain data members, but they can also contain functions as members.

An object is an instantiation of a class. In terms of variables, a class would be the type, and an object would be the variable.

Classes allow programming using object-oriented paradigms: Data and functions are both members of the object, reducing the need to pass and carry handlers or other state variables as arguments to functions, because they are part of the object whose member is called.

## Class definition

Classes are defined using either keyword `class` or keyword `struct`, with the following syntax:

```c++
class class_name {
  access_specifier_1:
    member1;
  access_specifier_2:
    member2;
} object_names;
```

where `class_name` is a valid identifier for the class, `object_names` is an optional list of names for objects of this class. The body of the declaration can contain members (**data or function declarations**), and optionally **access specifiers**.

## Classes defined with struct and union

Classes can be defined not only with keyword `class`, but also with keywords `struct` and `union`.

The keyword `struct`, generally used to declare plain data structures, can also be used to declare classes that have member functions, with the same syntax as with keyword `class`. The only difference between both is that members of classes declared with the keyword `struct` have `public` access by default, while members of classes declared with the keyword `class` have `private` access by default. For all other purposes both keywords are equivalent in this context.

Conversely, the concept of *unions* is different from that of classes declared with `struct` and `class`, since unions only store one data member at a time, but nevertheless they are also classes and can thus also hold member functions. The default access in union classes is `public`.  

## Members visibility

An **access specifier** is one of the following three keywords: `private`, `public` or `protected`. *These specifiers modify the access rights for the members that follow them*:

* `private` members of a class are accessible only from within other members of the same class (or from their *"friends"*).
* `protected` members are accessible from other members of the same class (or from their *"friends"*), but also from members of their *derived classes*.
* `public` members are accessible from anywhere where the object is visible.

By default, all members of a class declared with the `class` keyword have **private access** for all its members. Therefore, any member that is declared before any other *access specifier* has private access automatically.

For example:

```c++
class Rectangle {
    int width, height;  // default is private
  public:
    void set_values (int,int);
    int area (void);
} rect;
```

After the declarations of `Rectangle` and `rect`, any of the public members of object `rect` can be accessed as if they were normal functions or normal variables, by simply inserting a dot (`.`) between *object name* and *member name*. This follows the same syntax as accessing the members of plain data structures. For example: 

```c++
rect.set_values (3,4);
myarea = rect.area(); 
```

## Member function definition

For example:

```c++
class Rectangle {
    int width, height;
  public:
    void set_values (int,int);  // just declaration
    int area (void) { return width*height; }  // inline member function
} rect;

void Rectangle::set_values (int x, int y) {
  width = x;
  height = y;
}
```

Notice that the definition of the member function `area` has been included directly within the definition of class `Rectangle`given its extreme simplicity. Conversely, `set_values` it is merely declared with its prototype within the class, but its definition is outside it. In this outside definition, the operator of scope (`::`) is used to specify that the function being defined is a member of the class `Rectangle` and not a regular non-member function.

The scope operator (`::`) specifies the class to which the member being defined belongs, granting exactly the same scope properties as if this function definition was directly included within the class definition. For example, the function `set_values` in the previous example has access to the variables `width` and `height`, which are private members of class `Rectangle`, and thus only accessible from other members of the class, such as this.

The only difference between defining a member function completely within the class definition or to just include its declaration in the function and define it later outside the class, is that in the first case the function is automatically considered an **inline member function** by the compiler, while in the second it is a normal (not-inline) class member function. This causes no differences in behavior, but only on possible **compiler optimizations**.

## Class instances

The most important property of a class is that it is a type, and as such, we can declare multiple objects of it. 

```
Rectangle recta, rectb;
```

In this particular case, the class (type of the objects) is `Rectangle`, of which there are two instances (i.e., objects): `recta`and `rectb`. Each one of instances has its own member variables and member functions.

## Constructors

What would happen in the previous example if we called the member function `area` before having called `set_values`? An undetermined result, since the members `width` and `height` had never been assigned a value.

In order to avoid that, a class can include a special function called its **constructor**, which is automatically called whenever a new object of this class is created, allowing the class to *initialize member variables or allocate storage*.

```c++
class Rectangle {
    int width, height;
  public:
    Rectangle (int,int);
    int area () {return (width*height);}
};

Rectangle::Rectangle (int a, int b) {
  width = a;
  height = b;
}
```

 When we create a new object, the constructor will be automatically called:

```
Rectangle rect(2, 3);
```

Constructors cannot be called explicitly as if they were regular member functions. They are only executed once, when a new object of that class is created.

Notice how neither the constructor prototype declaration (within the class) nor the latter constructor definition, have return values; not even `void`: Constructors never return values, they simply initialize the object.  

## Overloading constructors

Like any other function, a constructor can also be overloaded with different versions taking different parameters: with a different number of parameters and/or parameters of different types. The compiler will automatically call the one whose parameters match the arguments:

```c++
class Rectangle {
    int width, height;
  public:
  	Rectangle();
    Rectangle(int, int);
    int area() { return width*height; }
};

Rectangle::Rectangle() {
    width = 5;
    height = 5;
}

Rectangle::Rectangle(int w, int h) {
    widht = w;
    height = h;
}
```

When we create a new object, the corresponding constructor will be called:

```c++
Rectangle recta;		// Rectangle::Rectangle() be called
Rectangle rectb(3, 4);  // Rectangle::Rectangle(int, int) be called
```

The above example also introduces a special kind constructor: the **default constructor**. The *default constructor* is the constructor that takes no parameters, and it is special because it is called when an object is declared but is not initialized with any arguments. In this example, the *default constructor* is called for `recta`. Note how `recta` is not even constructed with an empty set of parentheses - in fact, empty parentheses cannot be used to call the default constructor, it's considered as a function declaration:

```c++
Rectangle rectc(); // oops, default constructor NOT called, this is a function declaration
```

## Member initialization in constructors

When a constructor is used to initialize other members, these other members can be initialized directly, without resorting to statements in its body. This is done by inserting, before the constructor's body, a colon (`:`) and a list of initializations for class members. For example:

```c++
class Rectangle {
    int width,height;
  public:
    Rectangle(int,int);
    int area() {return width*height;}
};
```

We can initialize members in the body of constructor:

```c++
Rectangle::Rectangle(int w, int h) {
    // width and height already initialized before the function body run
	width = w;  // assign w to width
	height = h; // assign h to height
}
```

Or we can use a member initialization list for constructor:

```c++
Rectangle::Rectangle(int w, int h): width(x), height(y) { }
```

For members of fundamental types, it makes no difference which of the ways above the constructor is defined, because they are not initialized by default, but for member objects (those whose type is a class), if they are not initialized after the colon, they are default-constructed.

Default-constructing all members of a class may or may always not be convenient: in some cases, this is a waste (when the member is then reinitialized otherwise in the constructor), but in some other cases, default-construction is not even possible (when the class does not have a default constructor). In these cases, members shall be initialized in the member initialization list. For example:

```c++
// Circle has only on constructor, it isn't a default constructor.
class Circle {
    double radius;
  public:
    Circle(double r) : radius(r) { }
    double area() {return radius*radius*3.14159265;}
};

class Cylinder {
    Circle base;
    double height;
  public:
  	Cylinder(double, double);
  	double area() {return base.area()*height;}
};

// Cylinder::Cylinder(double r, double h) { base(r); height(h); }  // Error
// we must call Circle's non-default-constructor in the member initializer list
Cylinder::Cylinder(double r, double h): base(r), height(h) {}
// or
// Cylinder::Cylinder (double r, double h) : base{r}, height{h} { }
```

## Object uniform initialization

The way of calling constructors by enclosing their arguments in parentheses (`()`), as shown above, is known as **functional form**. But constructors can also be called with other syntaxes:

First, constructors with a single parameter can be called using the variable initialization syntax (an equal sign followed by the argument):

```
class_name object_name = initialization_value;
```

More recently, C++ introduced the possibility of constructors to be called using **uniform initialization**, which essentially is the same as the functional form, but using braces (`{}`) instead of parentheses (`()`):

```c++
class_name object_name { value1, value2, value3}
class_name object_name = { value1, value2, value3}
```

Optionally, this last syntax can include an equal sign before the braces.  

Here is an example with four ways to construct objects of a class whose constructor takes a single parameter:

```c++
// define class
class Circle {
    double radius;
  public:
    Circle(double r) { radius = r; }
    double circum() {return 2*radius*3.14159265;}
};

// create objects
Circle foo (10.0);   // functional form
Circle bar = 20.0;   // assignment init.
Circle baz {30.0};   // uniform init.
Circle qux = {40.0}; // POD-like
```

An advantage of uniform initialization over functional form is that, unlike parentheses, braces cannot be confused with function declarations, and thus can be used to explicitly call default constructors:

```
Rectangle rectb;   // default constructor called
Rectangle rectc(); // function declaration (default constructor NOT called)
Rectangle rectd{}; // default constructor called 
```

We can use uniform initialization to initialize a array of objects:

```c++
Rectangle bar(5, 6);  // a single object
Rectangle baz[2] { {2,5}, {3,6} };  // a array of objects
```

## Pointers to classes

Objects can also be pointed to by pointers: Once declared, a class becomes a valid type, so it can be used as the type pointed to by a pointer. For example: 

```
Rectangle * prect;
```

where `prect` is a pointer to an object of class `Rectangle`.

Similarly as with plain data structures, the members of an object can be accessed directly from a pointer by using the arrow operator (`->`).  For example:

```
Rectangle rect;
prect = &rect;
prect->area();
```

## Overloading operators

Classes, essentially, define new types to be used in C++ code. And types in C++ not only interact with code by means of constructions and assignments. They also interact by means of operators.

For a fundamental arithmetic type, the meaning of addition, subtraction, multiplication and assignment operations is generally obvious and unambiguous, but it may not be so for certain class types. For example:

```
struct myclass {
  string product;
  float price;
} a, b, c;
a = b + c;
```

Here, it is not obvious what the result of the addition operation on `b` and `c` does. In fact, this code alone would cause a compilation error, since the type `myclass` has no defined behavior for additions.

### Operators can be overloaded

However, C++ allows most operators to be overloaded so that their behavior can be defined for just about any type, including classes. Here is a list of all the operators that can be overloaded:

```
+    -    *    /    =    <    >    +=   -=   *=   /=   <<   >>
<<=  >>=  ==   !=   <=   >=   ++   --   %    &    ^    !    |
~    &=   ^=   |=   &&   ||   %=   []   ()   ,    ->*  ->   new 
delete    new[]     delete[]
```

### How to overload operators

Operators are overloaded by means of `operator` functions, which are regular functions with special names: their name begins by the `operator` keyword followed by the *operator sign* that is overloaded. The syntax is:

```
type operator sign (parameters) { /*... body ...*/ }
```

For example:

```c++
#include <iostream>
using namespace std;

class CVector {
  public:
    int x,y;
    CVector () {};
    CVector (int a,int b) : x(a), y(b) {}
    CVector operator + (const CVector&);
};

// overloads the addition operator
CVector CVector::operator+ (const CVector& param) {
  CVector temp;
  temp.x = x + param.x;
  temp.y = y + param.y;
  return temp;
}

int main () {
  CVector foo (3,1);
  CVector bar (1,2);
  CVector result;
  result = foo + bar;  // foo.operator+(bar);
  cout << result.x << ',' << result.y << '\n';
  return 0;
}
```

The operator overloads are just regular functions which can have any behavior; there is actually no requirement that the operation performed by that overload bears a relation to the mathematical or usual meaning of the operator, although it is strongly recommended.

### Parameters for operation functions

The parameter expected for a member function overload for operations such as `operator+` is naturally the operand to the right hand side of the operator. This is common to all binary operators (those with an operand to its left and one operand to its right). But operators can come in diverse forms. Here you have a table with a summary of the parameters needed for each of the different operators than can be overloaded (please, replace `@` by the operator in each case):

| Expression  | Operator                                        | Member function         | Non-member function |
| ----------- | ----------------------------------------------- | ----------------------- | ------------------- |
| `@a`        | `+ - * & ! ~ ++ --`                             | `A::operator@()`        | `operator@(A)`      |
| `a@`        | `++ --`                                         | `A::operator@(int)`     | `operator@(A,int)`  |
| `a@b`       | `+ - * / % ^ & | < > == != <= >= << >> && || ,` | `A::operator@(B)`       | `operator@(A,B)`    |
| `a@b`       | `= += -= *= /= %= ^= &= |= <<= >>= []`          | `A::operator@(B)`       | -                   |
| `a(b,c...)` | `()`                                            | `A::operator()(B,C...)` | -                   |
| `a->b`      | `->`                                            | `A::operator->()`       | -                   |
| `(TYPE) a`  | `TYPE`                                          | `A::operator TYPE()`    | -                   |

### Overload by non-member function

Notice that some operators may be overloaded in two forms: either as a member function or as a **non-member function**:

```c++
class CVector {
  public:
    int x,y;
    CVector () {}
    CVector (int a, int b) : x(a), y(b) {}
};

// overload by non-member function
CVector operator+ (const CVector& lhs, const CVector& rhs) {
  CVector temp;
  temp.x = lhs.x + rhs.x;
  temp.y = lhs.y + rhs.y;
  return temp;
}
```

## The keyword this

The keyword `this` represents a pointer to the object whose member function is being executed. It is used within a class's member function to refer to the object itself.

One of its uses can be to check if a parameter passed to a member function is the object itself. For example:  

```c++
class Dummy {
  public:
    bool isme(const Dummy& other);
}

bool Dummy::isme(const Dummy& other) {
    if (this == &other) return true;
    else return false;
}
```

It is also frequently used in `operator=` member functions that return objects by reference. For example:

```c++
CVector& CVector::operator= (const CVector& other) {
    //this->x = other.x;
    //this->y = other.y;
    x = other.x;
    y = other.y;
    return *this;
}
```

## Static members

A class can contain static members, either data or functions.

### Static data members

A **static data member** of a class is also known as a "**class variable**", because there is only one common variable for all the objects of that same class, sharing the same value: i.e., its value is not different from one object of this class to another. 

For example:

```c++
#include <iostream>
using namespace std;

class Dummy {
  public:
    // share with all of objects with this class
    static int n;
    Dummy () { n++; };
};

// static data members must be initialized outside the class
int Dummy::n=0;

int main () {
  Dummy a;
  Dummy b[5];
  // any object can refer the static data meber
  cout << a.n << '\n';
  Dummy * c = new Dummy;
  // or even directly by the class name
  cout << Dummy::n << '\n';
  delete c;
  return 0;
}
```

In fact, static members have the same properties as non-member variables but they enjoy class scope. For that reason, and to avoid them to be declared several times, they cannot be initialized directly in the class, but need to be initialized somewhere outside it.

### Static member functions

Classes can also have static member functions. These represent the same: members of a class that are common to all object of that class, acting exactly as non-member functions but being accessed like members of the class. Because they are like non-member functions, they cannot access non-static members of the class (neither member variables nor member functions). They neither can use the keyword `this`.

## Const members

When an object of a class is qualified as a `const` object:

```
const MyClass myobject;
```

The access to its data members from outside the class is restricted to read-only, as if all its data members were const for those accessing them from outside the class. Note though, that the constructor is still called and is allowed to initialize and modify these data members:

```c++
#include <iostream>
using namespace std;

class MyClass {
  public:
    int x;
    MyClass(int val) : x(val) {}
    int get() {return x;}
};

int main() {
  const MyClass foo(10);
  cout << foo.x << '\n';  // ok: data member x can be read
  // foo.x = 20;            // not valid: x cannot be modified
  // foo.get();				// not valid: non-const member function cannot be called via foo
  return 0;
}
```

The member functions of a `const` object can only be called if they are themselves specified as `const` members; in the example above, member `get` (which is not specified as `const`) cannot be called from `foo`. To specify that a member is a `const` member, the `const` keyword shall **follow the function prototype**, after the closing parenthesis for its parameters:

```
int get() const {return x;}
```

`const` objects are limited to access only member functions marked as `const`, but non-`const` objects are not restricted and thus can access both `const` and non-`const` member functions alike.

Member functions specified to be `const` cannot modify non-static data members nor call other non-`const` member functions. In essence, `const` members shall not modify the state of an object.

Const objects are very common. Most functions taking classes as parameters actually take them by `const` reference, and thus, these functions can only access their `const`members:

```c++
class MyClass {
	int x;
  public:
    MyClass(int v): x(v) { }
    const int get() const { return x; }
};

void print(const MyClass& c) {
	// Note: const object can only access const member function.
	// Thus the member get() must be a const function.
    cout << c.get();
}
```

Member functions can be overloaded on their constness: i.e., a class may have two member functions with identical signatures except that one is `const` and the other is not: in this case, the `const` version is called only when the object is itself const, and the non-const version is called when the object is itself non-const:

```c++
class MyClass {
	int x;
  public:
    MyClass(int v): x(v) { }
    // const member function
    const int get() const { return x; }
    // non-const member function
    const int get() { return x; }
};

int main() {
	MyClass c(3);
	cout << c.get();  // call the non-const get()
	const MyClass cc(5);
	cout << cc.get();  // call the const get()
	return 0;
}
```

## Class templates

Just like we can create function templates, we can also create class templates, allowing classes to have members that use template parameters as types. For example: 

```c++
template <class T>
class mypair {
    T values [2];
  public:
    mypair (T first, T second)
    {
      values[0]=first; values[1]=second;
    }
    T getmax ();
};

template <class T> 
T mypair::getmax() {
  T retval;
  retval = values[0]>values[1]? values[0] : values[1];
  return retval;    
}
```

We can use the template to declare objects as follows:

```c++
mypair<int> obj1(12, 34);
mypair<double> obj2(1.2, 3.4);
obj2.getmax();
```

### Template specialization

It is possible to define a different implementation for a template when a specific type is passed as template argument. This is called a **template specialization**.

For example, let's suppose that we have a very simple class called `mycontainer` that can store one element of any type and that has just one member function called `increase`, which increases its value. But we find that when it stores an element of type `char` it would be more convenient to have a completely different implementation with a function member `uppercase`, so we decide to declare a class template specialization for that type:  

```c++
// generic template:
template <class T>
class mycontainer {
    T element;
  public:
    mycontainer (T arg) {element=arg;}
    T increase () {return ++element;}
};

// specialization template
template <>
class mycontainer <char> {
    char element;
  public:
    mycontainer (char arg) {element=arg;}
    char uppercase ()
    {
      if ((element>='a')&&(element<='z'))
      element+='A'-'a';
      return element;
    }
};


int main() {
    mycontainer<int> myint (7);
  	mycontainer<char> mychar ('j');
    return 0;
}
```

**Note:** When we declare specializations for a template class, we must also define all its members, even those identical to the generic template class, because there is no "inheritance" of members from the generic template to the specialization.

# Special member functions

Special member functions are member functions that are implicitly defined as member of classes under certain circumstances. There are six:

| Member function                                              | typical form for class `C`: |
| ------------------------------------------------------------ | --------------------------- |
| [Default constructor](http://www.cplusplus.com/doc/tutorial/classes2/#default_constructor) | `C::C();`                   |
| [Destructor](http://www.cplusplus.com/doc/tutorial/classes2/#destructor) | `C::~C();`                  |
| [Copy constructor](http://www.cplusplus.com/doc/tutorial/classes2/#copy_constructor) | `C::C (const C&);`          |
| [Copy assignment](http://www.cplusplus.com/doc/tutorial/classes2/#copy_assignment) | `C& operator= (const C&);`  |
| [Move constructor](http://www.cplusplus.com/doc/tutorial/classes2/#move) | `C::C (C&&);`               |
| [Move assignment](http://www.cplusplus.com/doc/tutorial/classes2/#move) | `C& operator= (C&&);`       |

## Default constructor

The default constructor is the constructor called when objects of a class are declared, but are not initialized with any arguments.

If a class definition has no constructors, the compiler assumes the class to have an implicitly defined *default constructor*. Therefore, after declaring a class like this:

```c++
class Foo {
    int x;
  public:
    int get() {return x;}
};
```

The compiler assumes that `Foo` has a *default constructor*.

But as soon as a class has some constructor taking any number of parameters explicitly declared, the compiler no longer provides an implicit default constructor, and no longer allows the declaration of new objects of that class without arguments. For example:

```c++
class Foo {
    int x;
  public:
    Foo(int v): x(v) {}
    int get() {return x;}
};

// Foo f; // not valid: no default constructor
```





.