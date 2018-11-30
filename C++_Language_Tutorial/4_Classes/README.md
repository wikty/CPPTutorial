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

If a class definition has no constructors, the compiler assumes the class to have an **implicitly default constructor**. Therefore, after declaring a class like this:

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

Foo f; // not valid: no default constructor
```

Therefore, if objects of this class need to be constructed without arguments, the proper *default constructor* shall also be explicitly declared in the class. For example:

```c++
class Foo {
    int x;
  public:
    Foo(): x(0) {}
    Foo(int v): x(v) {}
    int get() {return x;}
};

Foo f; // valid: default constructor will be called
Foo f(2); // valid: constructor Foo(int v) will be called
```

## Destructor

Destructors fulfill the opposite functionality of constructors: They are responsible for the necessary cleanup needed by a class when its lifetime ends. A destructor is a member function very similar to a *default constructor*: it takes no arguments and returns nothing, not even `void`. It also uses the class name as its own name, but preceded with a tilde sign (`~`).

For example:

```c++
class Foo {
    string *ptr;
  public:
    Foo();
    Foo(const string&);
   	~Foo();
    const string& get() const { return *ptr; }
};

// default constructor
Foo::Foo(): ptr(new string) {}
// another constructor
Foo:Foo(const string& str): prt(new string(str)) {}
// destructor: be called automatically at the end of the object's life in charge of releasing this memory.
Foo::~Foo() { delete prt; }
```

## Copy constructor

When an object is passed a named object of its own type as argument, its *copy constructor* is invoked in order to construct a copy. A *copy constructor* is a constructor whose first parameter is of type *reference to the class* itself (possibly `const`qualified) and which can be invoked with a single argument of this type. For example , the signature of copy constructor may be like this:

```c++
MyClass::MyClass (const MyClass&);
```

If a class has no custom **copy** nor **move** constructors (or **assignments**) defined, an **implicit copy constructor** is provided. This copy constructor simply performs a copy of its own members. For example, for a class such as:

```c++
class MyClass {
  public:
    int a, b; string c;
};
```

An **implicit copy constructor** is automatically defined. The definition assumed for this function performs a **shallow copy**, roughly equivalent to:

```c++
MyClass::MyClass(const MyClass& other): a(other.a), b(other.b), c(other.c) {}
```

This default *copy constructor* may suit the needs of many classes. But *shallow copies* only copy the members of the class themselves, and this is probably not what we expect for classes like:

```c++
class Foo {
    string *ptr;
  public:
    Foo();
    Foo(const string&);
   	~Foo();
    const string& get() const { return *ptr; }
};

// the implicit copy constructor
Foo::Foo(const Foo& other): ptr(other.ptr) {}
```

The `Foo` class contains pointers of which it handles its storage. For that class, performing a *shallow copy* means that the pointer value is copied, but not the content itself; This means that both objects (the copy and the original) would be sharing a single `string` object (they would both be pointing to the same object), and at some point (on destruction) both objects would try to delete the same block of memory, probably causing the program to crash on runtime. This can be solved by defining the following **custom copy constructor** that performs a **deep copy**:

```c++
Foo::Foo(const Foo& other): ptr(new string(other.get())) {}
```

The *deep copy* performed by this *copy constructor* allocates storage for a new string, which is initialized to contain a copy of the original object. In this way, both objects (copy and original) have distinct copies of the content stored in different locations.

## Copy Assignment

Objects are not only copied on construction, when they are initialized: They can also be copied on any assignment operation. See the difference:

```c++
MyClass foo;
MyClass bar (foo);       // object initialization: copy constructor called
MyClass baz = foo;       // object initialization: copy constructor called
foo = bar;               // object already initialized: copy assignment called
```

Note that `baz` is initialized on construction using an *equal sign*, but this is not an assignment operation! (although it may look like one): The declaration of an object is not an assignment operation, it is just another of the syntaxes to call single-argument constructors. The assignment on `foo` is an assignment operation. No object is being declared here, but an operation is being performed on an existing object; `foo`.

The *copy assignment operator* is an overload of `operator=` which takes a *value* or *reference* of the class itself as parameter. The return value is generally a reference to `*this` (although this is not required). For example, the signature of copy assignment may be like this:

```c++
MyClass& MyClass::operator= (const MyClass&);
```

The *copy assignment operator* is also a *special function* and is also defined implicitly if a class has no custom **copy** nor **move** assignments (nor move constructor) defined. And the **implicit copy assignment** performs a **shallow copy**.

A **deep copy** version of custom copy assignment for the `Foo` class as follows:

```c++
Foo& Foo::operator= (const Foo& x) {
    delete ptr;                      // delete currently pointed string
  	ptr = new string (x.get());      // allocate space for new string, and copy
  	return *this;					 // return current object
}
```

## Move constructor and assignment

Similar to copying, moving also uses the value of an object to set the value to another object. But, unlike copying, the content is actually transferred from one object (the source) to the other (the destination): the source loses that content, which is taken over by the destination. This **moving** only happens when the **source** of the value is an **unnamed object**. 

*Unnamed objects* are objects that are temporary in nature, and thus haven't even been given a name. Typical examples of *unnamed objects* are return values of functions or type-casts.  

Using the value of a **temporary object** such as these to **initialize** another object or to **assign** its value, does not really require a copy: the object is never going to be used for anything else, and thus, its value can be *moved into* the destination object. These cases trigger the *move constructor* and *move assignments*:

```c++
MyClass fn();            // function returning a MyClass object
MyClass foo;             // default constructor

MyClass bar = foo;       // copy constructor
MyClass baz = fn();      // move constructor: called when an object is initialized on construction using an unnamed temporary

foo = bar;               // copy assignment
baz = MyClass();         // move assignment: called when an object is assigned the value of an unnamed temporary
```

The move constructor and move assignment are members that take a parameter of type **rvalue reference** to the class-itself:

```c++
MyClass (MyClass&&);             // move-constructor
MyClass& operator= (MyClass&&);  // move-assignment 
```

An *rvalue reference* is specified by following the type with two ampersands (`&&`). As a parameter, an *rvalue reference* matches arguments of temporaries of this type.

The concept of moving is most useful for objects that manage the storage they use, such as objects that allocate storage with new and delete. In such objects, **copying and moving** are really different operations:
- Copying from A to B means that new memory is allocated to B and then the entire content of A is copied to this new memory allocated for B.
- Moving from A to B means that the memory already allocated to A is transferred to B without allocating any new storage. It involves simply copying the pointer.

Example:

```c++
// move constructor
Foo::Foo(Foo&& other): ptr(other.ptr) { other.ptr = nullptr; }

// move assignment
Foo& Foo::operator= (Foo&& other) {
    delete ptr;
    ptr = other.ptr;
    other.ptr = nullptr;
    return *this;
}
```

Compilers already optimize many cases that formally require a move-construction call in what is known as *Return Value Optimization*. Most notably, when the value returned by a function is used to initialize an object. In these cases, the *move constructor* may actually never get called.

Note that even though *rvalue references* can be used for the type of any function parameter, it is seldom useful for uses other than the *move constructor*. Rvalue references are tricky, and unnecessary uses may be the source of errors quite difficult to track.  

## Implicit members functions

The six special members functions described above are members implicitly declared on classes under certain circumstances:

| Member function                                              | implicitly defined:                                          | default definition: |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------- |
| [Default constructor](http://www.cplusplus.com/doc/tutorial/classes2/#default_constructor) | if no other constructors                                     | does nothing        |
| [Destructor](http://www.cplusplus.com/doc/tutorial/classes2/#destructor) | if no destructor                                             | does nothing        |
| [Copy constructor](http://www.cplusplus.com/doc/tutorial/classes2/#copy_constructor) | if no move constructor and no move assignment                | copies all members  |
| [Copy assignment](http://www.cplusplus.com/doc/tutorial/classes2/#copy_assignment) | if no move constructor and no move assignment                | copies all members  |
| [Move constructor](http://www.cplusplus.com/doc/tutorial/classes2/#move) | if no destructor, no copy constructor and no copy nor move assignment | moves all members   |
| [Move assignment](http://www.cplusplus.com/doc/tutorial/classes2/#move) | if no destructor, no copy constructor and no copy nor move assignment | moves all members   |

Notice how not all special member functions are implicitly defined in the same cases. This is mostly due to backwards compatibility with C structures and earlier C++ versions, and in fact some include deprecated cases.

Fortunately, each class can select explicitly which of these members exist with their default definition or which are deleted by using the keywords `default` and `delete`, respectively. The syntax is either one of:

```
function_declaration = default;
function_declaration = delete;
```

For example:

```c++
class Foo {
  public:
  	Foo() = default;
  	Foo(const Foo&) = delete;
};
```

In general, and for future compatibility, classes that explicitly define one copy/move constructor or one copy/move assignment but not both, are encouraged to specify either `delete` or `default` on the other special member functions they don't explicitly define.

# Friendship

## Friend functions

In principle, private and protected members of a class cannot be accessed from outside the same class in which they are declared. However, this rule does not apply to *"friends"*.

A non-member function can access the private and protected members of a class if it is declared a *friend* of that class. That is done by including a declaration of this external function within the class, and preceding it with the keyword `friend`:

```c++
class Rectangle {
    int width, height;
  public:
    Rectangle() {}
    Rectangle (int x, int y) : width(x), height(y) {}
    int area() {return width * height;}
    // declare external function as friend function
    friend Rectangle duplicate (const Rectangle&);
};

// define the external friend function
Rectangle duplicate (const Rectangle& param)
{
  // Now we can access the private members of Rectangle
  Rectangle res;
  res.width = param.width*2;
  res.height = param.height*2;
  return res;
}
```

Typical use cases of friend functions are operations that are conducted between two different classes accessing private or protected members of both. 

## Friend classes

Similar to friend functions, a friend class is a class whose members have access to the private or protected members of another class:

```c++
class Square;

class Rectangle {
    double width, height;
  public:
  	Rectangle(double w, double h): width(w), height(h) {}
  	double area() { return width*height; }
  	void convert(Square s);
};

class Square {
  // declare friend class
  friend class Rectangle;
  private:
    double side;
  public:
    Square(double s): side(s) {}
    double area() { return side*side; }
};

// Rectangle's members can access Square private and protected members
void Rectangle::convert(Square s) {
    width = s.side;
    height = s.side;
}
```

Friendships are never corresponded unless specified: In our example, `Rectangle` is considered a friend class by `Square`, but Square is not considered a friend by `Rectangle`. Therefore, the member functions of `Rectangle` can access the protected and private members of `Square` but not the other way around. Of course, `Square` could also be declared friend of `Rectangle`, if needed, granting such an access.

Another property of friendships is that they are not transitive: The friend of a friend is not considered a friend unless explicitly specified.

# Inheritance

Classes in C++ can be extended, creating new classes which retain characteristics of the base class. This process, known as **inheritance**, involves a **base class** and a **derived class**: The *derived class* inherits the members of the *base class*, on top of which it can add its own members.

Classes that are derived from others inherit all the accessible members (protected and public members) of the base class. That means that if a base class includes a member `A` and we derive a class from it with another member called `B`, the derived class will contain both member `A` and member `B`.

## Syntax

The inheritance relationship of two classes is declared in the derived class. Derived classes definitions use the following syntax:

```c++
class derived_class_name: public base_class_name{ /*...*/ };
```

where `derived_class_name` is the name of the derived class and `base_class_name` is the name of the class on which it is based. The `public` access specifier may be replaced by any one of the other access specifiers (`protected` or `private`).  This access specifier limits the most accessible level for the members inherited from the base class: The members with a more accessible level are inherited with this level instead, while the members with an equal or more restrictive access level keep their restrictive level in the derived class.

With `protected`, all public members of the base class are inherited as `protected` in the derived class. Conversely, if the most restricting access level is specified (`private`), all the base class members are inherited as `private`.

If no access level is specified for the inheritance, the compiler assumes private for classes declared with keyword `class`and public for those declared with `struct`.

Actually, most use cases of inheritance in C++ should use public inheritance. When other access levels are needed for base classes, they can usually be better represented as member variables instead.  

## Example

The `Polygon` class would contain members that are common for both types of polygon.  And `Rectangle` and `Triangle` would be its derived classes, with specific features that are different from one type of polygon to the other:

```c++
class Polygon {
  // derived classes can access protected members
  protected:
    double width, height;
  public:
    void set_values(double w, double h){
        widht = w;
        height = h;
    }
};

class Rectangle: public Polygon {
  public:
  	double area() { return width*height; }
};

class Triangle: public Polygon {
  public:
  	double area() { return (width*height)/2; }
};
```

The `protected` access specifier used in class `Polygon` is similar to `private`. Its only difference occurs in fact with inheritance: When a class inherits another one, the members of the derived class can access the protected members inherited from the base class, but not its private members.

This `public` keyword after the colon (`:`) denotes the most accessible level the members inherited from the class that follows it (in this case `Polygon`) will have from the derived class (in this case `Rectangle`). Since `public` is the most accessible level, by specifying this keyword the derived class will inherit all the members with the same levels they had in the base class.

## Multiple inheritance

A class may inherit from more than one class by simply specifying more base classes, separated by commas, in the list of a class's base classes (i.e., after the colon). For example:

```c++
class BaseOne {};
class BaseTwo {};

class Derived: public BaseOne, public BaseTwo {};
```

## What is inherited from the base class?

In principle, a publicly derived class inherits access to every member of a base class except:

- its constructors and its destructor
- its assignment operator members (operator=)
- its friends
- its private members

## Automatically call base class constructor

Even though access to the constructors and destructor of the base class is not inherited as such, they are automatically called by the constructors and destructor of the derived class.

Unless otherwise specified, the constructors of a derived class calls the default constructor of its base classes (i.e., the constructor taking no arguments). Calling a different constructor of a base class is possible, using the same syntax used to initialize member variables in the initialization list:

```c++
derived_constructor_name (parameters) : base_constructor_name (parameters) {...}
```

For example:

```c++
class Polygon {
  // derived classes can access protected members
  protected:
    double width, height;
  public:
    // default constructor
    Polygon(): widht(0.0), height(0.0) {}
    // another constructor
  	Polygon(double w, double h): width(w), height(h) {}
};

class Rectangle: public Polygon {
  public:
    // automatically call the default constructor of base class
  	Rectangle(double w, double h) {}
  	double area() { return width*height; }
};

class Triangle: public Polygon {
  public:
    // call the another constructor of base class
  	Triangle(double w, double h): Polygon(w, h) {}
  	double area() { return (width*height)/2; }
};
```

# Polymorphism

One of the key features of class inheritance is that a pointer to a derived class is type-compatible with a pointer to its base class. *Polymorphism* is the art of taking advantage of this simple but powerful and versatile feature. Virtual members and abstract classes grant C++ polymorphic characteristics, most useful for object-oriented projects. 

## Base class pointers access derived class

For example, we have a base class `Polygon` and two derived class `Rectangle` and `Triangle`:

```c++
class Polygon {
  // derived classes can access protected members
  protected:
    double width, height;
  public:
    void set_values(double w, double h){
        widht = w;
        height = h;
    }
};

class Rectangle: public Polygon {
  public:
  	double area() { return width*height; }
};

class Triangle: public Polygon {
  public:
  	double area() { return (width*height)/2; }
};
```

Then we can do something as follows:

````c++
Rectangle rect;
Triangle trgl;
Polygon * ppoly1 = &rect;
Polygon * ppoly2 = &trgl;

// use base class pointer to manipulate derived class
ppoly1->set_values(4,5);
ppoly2->set_values(4,5);

// use derived class to access members belong to itself
cout << "Rectangle area: " << rect.area();
cout << "Triangle area: " << trgl.area();
````

Because the type of `ppoly1` and `ppoly2` is pointer to `Polygon` (and not pointer to `Rectangle` nor pointer to `Triangle`), only the members inherited from `Polygon` can be accessed, and not those of the derived classes `Rectangle` and `Triangle`. That is why the program above accesses the `area` members of both objects using `rect` and `trgl` directly, instead of the pointers; the pointers to the base class cannot access the `area` members.

Member `area` could have been accessed with the pointers to `Polygon` if `area` were a member of `Polygon` instead of a member of its derived classes, but the problem is that `Rectangle` and `Triangle` implement different versions of `area`, therefore there is not a single common version that could be implemented in the base class. 

## Virtual members

*A virtual member is a member function that can be redefined in a derived class, while preserving its calling properties through references*. The syntax for a function to become virtual is to precede its declaration with the `virtual` keyword:

```c++
class Polygon {
  protected:
    double width, height;
  public:
    void set_values(double w, double h){
        widht = w;
        height = h;
    }
    virtual double area() { return 0.0; }
};

class Rectangle: public Polygon {
  public:
  	double area() { return width*height; }
};

class Triangle: public Polygon {
  public:
  	double area() { return (width*height)/2; }
};
```

Now we can use a pointer of `Polygon` to call the corresponding `area` member for derived class `Rectangle` and `Triangle`:

```c++
Rectangle rect;
Triangle trgl;
Polygon * ppoly1 = &rect;
Polygon * ppoly2 = &trgl;

ppoly1->set_values(4,5);
ppoly2->set_values(4,5);
cout << "Rectangle area: " << ppoly1->area();
cout << "Triangle area: " << ppoly2->area();
```

The member function `area` has been declared as `virtual` in the base class `Polygon` because it is later redefined in each of the derived classes `Rectangle` and `Triangle`. 

**Note:** Non-virtual members can also be redefined in derived classes, but non-virtual members of derived classes cannot be accessed through a reference of the base class: i.e., if `virtual` is removed from the declaration of `area`in the example above, all calls to `area` would return zero, because in all cases, the version of the base class would have been called instead.

Therefore, essentially, what the `virtual` keyword does is to allow a member of a derived class with the same name as one in the base class to be appropriately called from a pointer, and more precisely when the type of the pointer is a pointer to the base class that is pointing to an object of the derived class, as in the above example. A class that declares or inherits a virtual function is called a **polymorphic class**.

## Abstract base classes

Note that, in the above example, despite of the virtuality of one of its members, `Polygon` was a regular class, of which even an object was instantiated (`poly`), with its own definition of member `area` that always returns 0. And `Polygen poly;` is allowed.

**Abstract base classes** are something very similar to the `Polygon` class in the previous example. But they are classes that can only be used as base classes, and thus are allowed to have virtual member functions without definition (known as **pure virtual functions**). The syntax is to replace their definition by `=0` (an equal sign and a zero). The abstract base class version of `Polygon` as follows:

```c++
class Polygon {
  protected:
    double width, height;
  public:
    void set_values(double w, double h){
        widht = w;
        height = h;
    }
    // pure virtual function
    virtual double area() = 0;
};
```

Notice that `area` has no definition; this has been replaced by `=0`, which makes it a *pure virtual function*. Classes that contain at least one **pure virtual function** are known as **abstract base classes**. Abstract base classes cannot be used to instantiate objects, i.e. `Polygon poly;` is not allowed.

Why we need abstract base classes? Abstract base classes can be used to create pointers to it, and take advantage of all its polymorphic abilities. And can actually be dereferenced when pointing to objects of derived (non-abstract) classes. It is even possible for a member of the abstract base class `Polygon` to use the special pointer `this` to access the proper virtual members, even though `Polygon` itself has no implementation for this function. For example:

```c++
class Polygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b)
      { width=a; height=b; }
    virtual int area() =0;
    void printarea(){ 
        // member area() will be implemented in derived classes
        cout << this->area() << '\n'; 
    }
};

class Rectangle: public Polygon {
  public:
    int area (void)
      { return (width * height); }
};

class Triangle: public Polygon {
  public:
    int area (void)
      { return (width * height / 2); }
};

int main() {
    Rectangle rect;
    Triangle trgl;
    Polygon * ppoly1 = &rect;
    Polygon * ppoly2 = &trgl;

    ppoly1->set_values(4,5);
    ppoly2->set_values(4,5);  
    ppoly1->printarea();
    ppoly2->printarea();
}
```

In this example, objects of different but related types are referred to using a unique type of pointer (`Polygon*`) and the proper member function is called every time, just because they are virtual.



