[TOC]



# Type conversions

## Implicit conversion

Implicit conversions are automatically performed when a value is copied to a compatible type. 

* For fundamental data types, this is known as a **standard conversion**. Standard conversions affect fundamental data types, and allow the conversions between numerical types (`short` to `int`, `int` to `float`, `double` to `int`...), to or from `bool`, and some pointer conversions.
  * Converting to `int` from some smaller integer type, or to `double` from `float` is known as **promotion**, and is guaranteed to produce the exact same value in the destination type.
  *  Other conversions between arithmetic types may not always be able to represent the same value exactly. Some of these conversions may imply a loss of precision, which the compiler can signal with a warning. This warning can be avoided with an explicit conversion.
* For non-fundamental types, arrays and functions implicitly convert to pointers, and pointers in general allow the following conversions:
  - *Null pointers* can be converted to pointers of any type
  - Pointers to any type can be converted to `void` pointers.
  - Pointer *upcast*: pointers to a derived class can be converted to a pointer of an *accessible* and *unambiguous* base class, without modifying its `const` or `volatile` qualification.

### Implicit conversions with classes

In the world of classes, implicit conversions can be controlled by means of three member functions:

- **Single-argument constructors:** allow implicit conversion from a particular type to initialize an object.
- **Assignment operator:** allow implicit conversion from a particular type on assignments.
- **Type-cast operator:** allow implicit conversion to a particular type.

For example:

```c++
class A { };
class B {
  public:
    // conversion from A (constructor)
    B(const A& a) {
        /* do some conversion jobs */
    }
    
    // conversion from A (assignment)
    B& operator=(const A& a) {
        /* do some conversion jobs */
        return *this;  // return an instance of B
    }
    
    // conversion to A (type-cast operator)
    operator A() { 
        /* do some conversion jobs */
        return A(); // return an instance of A
    }
};
```

### Prevents implicit conversions

Assume we have a function `void fn(B arg);` , an instance `a` of `A` and an instance `b` of `B` class that are defined by the above example. This function can be called with `a` as argument, there is a implicit conversion from type `A` to type `B`, i.e. `fn((B arg(a)))`.

It can be prevented by marking the affected constructor with the `explicit` keyword:

```c++
class A { };
class B {
  public:
    // conversion from A (constructor)
    explicit B(const A& a) {
        /* do some conversion jobs */
    }
}

void fn(B arg){ }
```

Now implicitly create instance of `B` from an instance of `A` is NOT allowed:

```c++
A a;
B b(a); // explicit conversion is allowed
fn(a);  // implicit conversion is not allowed
```

Type-cast member functions (those described in the previous section) can also be specified as `explicit`. This prevents implicit conversions in the same way as `explicit`-specified constructors do for the destination type.

## Explicit conversion

### type-casting

C++ is a strong-typed language. Many conversions, specially those that imply a different interpretation of the value, require an explicit conversion, known in C++ as **type-casting**. There exist two main syntaxes for generic type-casting: *functional* and *c-like*:

```c++
double x = 10.3;
int y;
y = int (x);    // functional notation: new_type (expression)
y = (int) x;    // c-like cast notation: (new_type) expression
```

The functionality of these generic forms of type-casting is enough for most needs with fundamental data types. However, these operators can be applied indiscriminately on classes and pointers to classes, which can lead to code that while being syntactically correct can cause **runtime errors**. For example, the following code **compiles without errors**: 

```c++
#include <iostream>
using namespace std;

class Dummy {
    double i,j;
};

class Addition {
    int x,y;
  public:
    Addition (int a, int b) { x=a; y=b; }
    int result() { return x+y;}
};

int main () {
  Dummy d;
  Addition * padd;
  padd = (Addition*) &d;
  cout << padd->result();
  return 0;
}
```

*Unrestricted explicit type-casting allows to convert any pointer into any other pointer type, independently of the types they point to*. The subsequent call to member `result` will produce either a runtime error or some other unexpected results.

In order to control these types of conversions between classes, we have four specific casting operators: `dynamic_cast`, `reinterpret_cast`, `static_cast` and `const_cast`. Their format is to follow the new type enclosed between angle-brackets (`<>`) and immediately after, the expression to be converted between parentheses, e.g. `static_cast <new_type> (expression)`.

### dynamic_cast

`dynamic_cast` can only be used with pointers and references to classes (or with `void*`). Its purpose is to ensure that the result of the type conversion points to a valid complete object of the destination pointer type.

This naturally includes **pointer upcast** (converting from pointer-to-derived to pointer-to-base), in the same way as allowed as an **implicit conversion**. But `dynamic_cast` can also **downcast** (convert from pointer-to-base to pointer-to-derived) polymorphic classes (those with virtual members) if and only if the pointed object is a valid complete object of the target type. For example:  

```c++
#include <iostream>
#include <exception>
using namespace std;

class Base { virtual void dummy(){} };
class Derived: public Base { int a; }

int main() {
    Base* pba = new Derived;
    Base* pbb = new Base;
    Derived* pd;
    
    try {
        // success: pba is pointing to a full object of class Derived
        pd = dynamic_cast<Derived*>(pba);
        if (pd == 0) { cout << "Null pointer on first type-cast.\n"; }
        
        // fail: pbb is pointing to a Base instance, it's an incomplete object of class Derived.
        pd = dynamic_cast<Derived*>(pbb);
        if (pd == 0) { cout << "Null pointer on second type-cast.\n"; }
    } catch (exception& e) {cout << "Exception: " << e.what();}
    
    delete pba;
    delete pbb;
    return 0;
}
```

When `dynamic_cast` cannot cast a pointer because it is not a complete object of the required class -as in the second conversion in the previous example- it returns a **null pointer** to indicate the failure. If `dynamic_cast` is used to convert to a reference type and the conversion is not possible, an exception of type `bad_cast` is thrown instead.

`dynamic_cast` can also perform the other implicit casts allowed on pointers: casting null pointers between pointers types (even between unrelated classes), and casting any pointer of any type to a `void*` pointer.

**Note:**  This type of `dynamic_cast` requires *Run-Time Type Information (RTTI)* to keep track of dynamic types. Some compilers support this feature as an option which is disabled by default. This needs to be enabled for runtime type checking using `dynamic_cast` to work properly with these types.

### static_cast

`static_cast` can perform conversions between pointers to **related classes**, not only **upcasts** (from pointer-to-derived to pointer-to-base), but also **downcasts** (from pointer-to-base to pointer-to-derived). *No checks are performed during runtime to guarantee that the object being converted is in fact a full object of the destination type*. Therefore, it is up to the programmer to ensure that the conversion is safe. On the other side, it does not incur the overhead of the type-safety checks of `dynamic_cast`.

```c++
Base* a = new Base;
// conversion success, but could lead to runtime errors
Derived* b = static_cast<Derived*>(a);
```

This would be valid code, although `b` would point to an incomplete object of the class and could lead to runtime errors if dereferenced.

`static_cast` is also able to perform all conversions allowed implicitly (not only those with pointers to classes), and is also able to perform the opposite of these. It can:

- Convert from `void*` to any pointer type. In this case, it guarantees that if the `void*` value was obtained by converting from that same pointer type, the resulting pointer value is the same.
- Convert integers, floating-point values and enum types to enum types.

Additionally, `static_cast` can also perform the following:

- Explicitly call a single-argument constructor or a conversion operator.
- Convert to *rvalue references*.
- Convert `enum class` values into integers or floating-point values.
- Convert any type to `void`, evaluating and discarding the value.

### reinterpret_cast

`reinterpret_cast` converts any pointer type to any other pointer type, even of **unrelated classes**. The operation result is a simple binary copy of the value from one pointer to the other. *All pointer conversions are allowed: neither the content pointed nor the pointer type itself is checked*.

It can also cast pointers to or from integer types. The format in which this integer value represents a pointer is platform-specific. The only guarantee is that a pointer cast to an integer type large enough to fully contain it (such as `intptr_t`), is guaranteed to be able to be cast back to a valid pointer. For example:

```c++
#include <cstdint>

class A {};

int main() {
    A* pa = new A;
    intptr_t i;
    
    i = reinterpret_cast<intptr_t>(pa);
    pa = reinterpret_cast<A*>(i);
    
    delete pa;
    return 0;
}
```

The conversions that can be performed by `reinterpret_cast` but not by `static_cast` are low-level operations based on reinterpreting the binary representations of the types, which on most cases results in code which is system-specific, and thus non-portable. For example:

```
class A { /* ... */ };
class B { /* ... */ };
A * a = new A;
B * b = reinterpret_cast<B*>(a);
```

### const_cast

This type of casting manipulates the constness of the object pointed by a pointer, either to be set or to be removed. For example, in order to pass a const pointer to a function that expects a non-const argument:

```c++
#include <iostream>
using namespace std;

void print (char * str)
{
  cout << str << '\n';
}

int main () {
  const char * c = "sample text";
  // remove constness of c
  print ( const_cast<char *> (c) );
  return 0;
}
```

The example above is guaranteed to work because function `print` does not write to the pointed object. Note though, that removing the constness of a pointed object to actually write to it causes *undefined behavior*.

## Check type

`typeid` allows to check the type of an expression: `typeid (expression)`.
This operator returns a reference to a constant object of type `type_info` that is defined in the standard header `<typeinfo>`. A value returned by `typeid` can be compared with another value returned by `typeid` using operators `==` and `!=`or can serve to obtain a null-terminated character sequence representing the data type or class name by using its `name()` member.  

```c++
#include <iostream>
#include <typeinfo>
using namespace std;

int main () {
  int * a,b;
  a=0; b=0;
  if (typeid(a) != typeid(b))
  {
    cout << "a and b are of different types:\n";
    cout << "a is: " << typeid(a).name() << '\n';
    cout << "b is: " << typeid(b).name() << '\n';
  }
  return 0;
}
```

When `typeid` is applied to classes, `typeid` uses the RTTI to keep track of the type of dynamic objects. When `typeid` is applied to an expression whose type is a polymorphic class, the result is the type of the most derived complete object:

```c++
#include <iostream>
#include <typeinfo>
#include <exception>
using namespace std;

class Base { virtual void f(){} };
class Derived : public Base {};

int main () {
  try {
    Base* a = new Base;
    Base* b = new Derived;
    cout << "a is: " << typeid(a).name() << '\n';  // class Base*
    cout << "b is: " << typeid(b).name() << '\n';  // class Base*
    cout << "*a is: " << typeid(*a).name() << '\n'; // class Base
    cout << "*b is: " << typeid(*b).name() << '\n';  // class Derived
  } catch (exception& e) { cout << "Exception: " << e.what() << '\n'; }
  return 0;
}
```

Notice how the type that `typeid` considers for pointers is the pointer type itself (both `a` and `b` are of type `class Base *`). However, when `typeid` is applied to objects (like `*a` and `*b`) `typeid` yields their dynamic type (i.e. the type of their most derived complete object).

If the type `typeid` evaluates is a pointer preceded by the dereference operator (`*`), and this pointer has a null value, `typeid` throws a `bad_typeid` exception.  

# Exceptions

**Exceptions** provide a way to react to exceptional circumstances (like runtime errors) in programs by transferring control to special functions called **handlers**.

To catch exceptions, a portion of code is placed under exception inspection. This is done by enclosing that portion of code in a **try-block**. *When an exceptional circumstance arises within that block, an exception is thrown that transfers the control to the exception handler. If no exception is thrown, the code continues normally and all handlers are ignored*.

## Syntax

An exception is thrown by using the `throw` keyword from inside the `try` block. Exception handlers are declared with the keyword `catch`, which must be placed immediately after the `try` block:  

```c++
#include <iostream>
using namespace std;

int main () {
  try
  {
    // throw exception with an integer as argument
    throw 20;
    // the following codes never been executed
    1+2;
  }
  catch (int e)
  {
    cout << "An exception occurred. Exception Nr. " << e << '\n';
  }
  catch (float e)
  {
      /* obviously, this handler never be matched */
  }
  catch (...)
  {
      /* this is a default handler that catches all exceptions not caught by other handlers */
  }
    
  /* program execution resumes from here after exception has been handled */
  3+4;  
  
  return 0;
}
```

 A `throw` expression accepts one parameter (in this case the integer value `20`), which is passed as an argument to the exception handler.

The exception handler is declared with the `catch` keyword immediately after the closing brace of the `try` block. The syntax for `catch` is similar to a regular function with one parameter. The type of this parameter is very important, since the type of the argument passed by the `throw` expression is checked against it, and only in the case they match, the exception is caught by that handler.

Multiple handlers (i.e., `catch` expressions) can be chained; each one with a different parameter type. Only the handler whose argument type matches the type of the exception specified in the `throw` statement is executed.  

If an ellipsis (`...`) is used as the parameter of `catch`, that handler will catch any exception no matter what the type of the exception thrown. This can be used as a default handler that catches all exceptions not caught by other handlers.

After an exception has been handled the program, execution resumes after the *try-catch* block, not after the `throw`statement!.

## Nest try-catch blocks

It is also possible to nest `try-catch` blocks within more external `try` blocks. In these cases, we have the possibility that an internal `catch` block forwards the exception to its external level. This is done with the expression `throw;` with no arguments. For example: 

```c++
try {
  try {
      // throw an exception
      throw 20;
  }
  catch (int n) {
  	  // the exception been throwed again 
      throw;
  }
}
catch (...) {
  cout << "Exception occurred";
}
```

## Exception specification

Older code may contain **dynamic exception specifications**. They are now deprecated in C++, but still supported. A *dynamic exception specification* follows the declaration of a function, appending a `throw` specifier to it. For example:

```
double myfunction (char param) throw (int);
```

This declares a function called `myfunction`, which takes one argument of type `char` and returns a value of type `double`. If this function throws an exception of some type other than `int`, the function calls [std::unexpected](http://www.cplusplus.com/unexpected) instead of looking for a handler or calling [std::terminate](http://www.cplusplus.com/terminate).

If this `throw` specifier is left empty with no type, this means that [std::unexpected](http://www.cplusplus.com/unexpected) is called for any exception. Functions with no `throw` specifier (regular functions) never call [std::unexpected](http://www.cplusplus.com/unexpected), but follow the normal path of looking for their exception handler.  

```c++
int myfunction (int param) throw(); // all exceptions call unexpected
int myfunction (int param);         // normal exception handling 
```

## Standard exceptions

The C++ Standard library provides a base class specifically designed to declare objects to be thrown as exceptions. It is called `std::exception` and is defined in the `<exception>` header. 

### Custom exceptions

The class `exception` has a virtual member function called `what` that returns a null-terminated character sequence (of type `char *`) and that can be overwritten in derived classes to contain some sort of description of the exception.

```c++
#include <iostream>
#include <exception>
using namespace std;

class myexception: public exception
{
  virtual const char* what() const throw()
  {
    return "My exception happened";
  }
} myex;

int main () {
  try
  {
    throw myex;
  }
  catch (exception& e)
  {
    cout << e.what() << '\n';
  }
  return 0;
}
```

We have placed a handler that catches exception objects by reference (notice the ampersand `&` after the type), therefore this catches also classes derived from `exception`, like our `myex` object of type `myexception`.

Also deriving from `exception`, header `<exception>` defines two generic exception types that can be inherited by custom exceptions to report errors:

| exception       | description                                        |
| --------------- | -------------------------------------------------- |
| `logic_error`   | error related to the internal logic of the program |
| `runtime_error` | error detected during runtime                      |

### Common standard exceptions

All exceptions thrown by components of the C++ Standard library throw exceptions derived from this `exception` class. These are:

| exception           | description                                              |
| ------------------- | -------------------------------------------------------- |
| `bad_alloc`         | thrown by `new` on allocation failure                    |
| `bad_cast`          | thrown by `dynamic_cast` when it fails in a dynamic cast |
| `bad_exception`     | thrown by certain dynamic exception specifiers           |
| `bad_typeid`        | thrown by `typeid`                                       |
| `bad_function_call` | thrown by empty `function` objects                       |
| `bad_weak_ptr`      | thrown by `shared_ptr` when passed a bad `weak_ptr`      |

A typical example where standard exceptions need to be checked for is on memory allocation:

```c++
#include <iostream>
#include <exception>
using namespace std;

int main () {
  try
  {
    int* myarray= new int[1000];
  }
  catch (exception& e)
  {
    cout << "Standard exception: " << e.what() << endl;
  }
  return 0;
}
```

The exception that may be caught by the exception handler in this example is a `bad_alloc`. Because `bad_alloc` is derived from the standard base class `exception`, it can be caught (capturing by reference, captures all related classes).

# Preprocessor directives

  Preprocessor directives are lines included in the code of programs preceded by a hash sign (`#`). These lines are not program statements but directives for the *preprocessor*. The preprocessor examines the code before actual compilation of code begins and resolves all these directives before any code is actually generated by regular statements.

These *preprocessor directives* extend only across a single line of code. As soon as a newline character is found, the preprocessor directive is ends. No semicolon (`;`) is expected at the end of a preprocessor directive. The only way a preprocessor directive can extend through more than one line is by preceding the newline character at the end of the line by a backslash (`\`).  

## Macro definitions

To define preprocessor macros we can use `#define`. Its syntax is:

```
#define identifier replacement
```

When the preprocessor encounters this directive, it replaces any occurrence of `identifier` in the rest of the code by `replacement`. This `replacement` can be an expression, a statement, a block or simply anything. The preprocessor does not understand C++ proper, it simply replaces any occurrence of `identifier` by `replacement`.  

Defined macros are not affected by block structure. A macro lasts until it is undefined with the `#undef` preprocessor directive:

```c++
#define TABLE_SIZE 100
int table1[TABLE_SIZE];  // 100
#undef TABLE_SIZE
#define TABLE_SIZE 200
int table2[TABLE_SIZE];  // 200
```

`#define` can work also with parameters to define function macros:

```
#define getmax(a,b) a>b?a:b 
```

This would replace any occurrence of `getmax` followed by two arguments by the replacement expression, but also replacing each argument by its identifier, exactly as you would expect if it was a function.

Function macro definitions accept two special operators (`#` and `##`) in the replacement sequence:
The operator `#`, followed by a parameter name, is replaced by a string literal that contains the argument passed (as if enclosed between double quotes):

```c++
#define str(x) #x
cout << str(test); // cout << "test";
```

The operator `##` concatenates two arguments leaving no blank spaces between them:

```c++
#define glue(a,b) a ## b
glue(co,ut) << "test";  // cout << "test";
```

Because preprocessor replacements happen before any C++ syntax check, macro definitions can be a tricky feature. But, be careful: code that relies heavily on complicated macros become less readable, since the syntax expected is on many occasions different from the normal expressions programmers expect in C++.

## Conditional inclusions

These directives allow to include or discard part of the code of a program if a certain condition is met.

`#ifdef` allows a section of a program to be compiled only if the macro that is specified as the parameter has been defined, no matter which its value is. For example:   

```c++
#ifdef TABLE_SIZE
int table[TABLE_SIZE];
#endif  
```

`#ifndef` serves for the exact opposite: the code between `#ifndef` and `#endif` directives is only compiled if the specified identifier has not been previously defined. For example:

```c++
#ifndef TABLE_SIZE
#define TABLE_SIZE 100
#endif
int table[TABLE_SIZE];
```

The `#if`, `#else` and `#elif` (i.e., "else if") directives serve to specify some condition to be met in order for the portion of code they surround to be compiled. The condition that follows `#if` or `#elif` can only evaluate constant expressions, including macro expressions. For example: 

```c++
#if TABLE_SIZE>200

#undef TABLE_SIZE
#define TABLE_SIZE 200
 
#elif TABLE_SIZE<50

#undef TABLE_SIZE
#define TABLE_SIZE 50
 
#else

#undef TABLE_SIZE
#define TABLE_SIZE 100

#endif
 
int table[TABLE_SIZE]; 
```

The behavior of `#ifdef` and `#ifndef` can also be achieved by using the special operators `defined` and `!defined` respectively in any `#if` or `#elif` directive:

```c++
#if defined ARRAY_SIZE
#define TABLE_SIZE ARRAY_SIZE
#elif !defined BUFFER_SIZE
#define TABLE_SIZE 128
#else
#define TABLE_SIZE BUFFER_SIZE
#endif 
```

## Line control

When we compile a program and some error happens during the compiling process, the compiler shows an error message with references to the name of the file where the error happened and a line number, so it is easier to find the code generating the error.

The `#line` directive allows us to control both things, the line numbers within the code files as well as the file name that we want that appears when an error takes place. Its format is: 

```
#line number "filename"
```

Where `number` is the new line number that will be assigned to the next code line. The line numbers of successive lines will be increased one by one from this point on.  

## Error directive

This directive aborts the compilation process when it is found, generating a compilation error that can be specified as its parameter:

```
#ifndef __cplusplus
#error A C++ compiler is required!
#endif 
```

This example aborts the compilation process if the macro name `__cplusplus` is not defined (this macro name is defined by default in all C++ compilers).

## Include directive

This directive has been used assiduously in other sections of this tutorial. When the preprocessor finds an `#include`directive it replaces it by the entire content of the specified header or file. There are two ways to use `#include`: 

```c++
#include <header>
#include "file" 
```

In the first case, a *header* is specified between angle-brackets `<>`. This is used to include headers provided by the implementation, such as the headers that compose the standard library (`iostream`, `string`,...). Whether the headers are actually files or exist in some other form is *implementation-defined*, but in any case they shall be properly included with this directive.

The syntax used in the second `#include` uses quotes, and includes a *file*. The *file* is searched for in an *implementation-defined* manner, which generally includes the current path. In the case that the file is not found, the compiler interprets the directive as a *header* inclusion, just as if the quotes (`""`) were replaced by angle-brackets (`<>`).  

## Pragma directive

This directive is used to specify diverse options to the compiler. These options are specific for the platform and the compiler you use. Consult the manual or the reference of your compiler for more information on the possible parameters that you can define with `#pragma`.

If the compiler does not support a specific argument for `#pragma`, it is ignored - no syntax error is generated.  

## Predefined macro names

The following macro names are always defined (they all begin and end with two underscore characters:

| macro             | value                                                        |
| ----------------- | ------------------------------------------------------------ |
| `__LINE__`        | Integer value representing the current line in the source code file being compiled. |
| `__FILE__`        | A string literal containing the presumed name of the source file being compiled. |
| `__DATE__`        | A string literal in the form "Mmm dd yyyy" containing the date in which the compilation process began. |
| `__TIME__`        | A string literal in the form "hh:mm:ss" containing the time at which the compilation process began. |
| `__cplusplus`     | An integer value. All C++ compilers have this constant defined to some value. Its value depends on the version of the standard supported by the compiler: **199711L**: ISO C++ 1998/2003**201103L**: ISO C++ 2011Non conforming compilers define this constant as some value at most five digits long. Note that many compilers are not fully conforming and thus will have this constant defined as neither of the values above. |
| `__STDC_HOSTED__` | `1` if the implementation is a *hosted implementation* (with all standard headers available) `0` otherwise. |

The following macros are optionally defined, generally depending on whether a feature is available:

| macro                              | value                                                        |
| ---------------------------------- | ------------------------------------------------------------ |
| `__STDC__`                         | In C: if defined to `1`, the implementation conforms to the C standard. In C++: Implementation defined. |
| `__STDC_VERSION__`                 | In C: **199401L**: ISO C 1990, Ammendment 1**199901L**: ISO C 1999**201112L**: ISO C 2011In C++: Implementation defined. |
| `__STDC_MB_MIGHT_NEQ_WC__`         | `1` if multibyte encoding might give a character a different value in character literals |
| `__STDC_ISO_10646__`               | A value in the form `yyyymmL`, specifying the date of the Unicode standard followed by the encoding of `wchar_t` characters |
| `__STDCPP_STRICT_POINTER_SAFETY__` | `1` if the implementation has *strict pointer safety* (see `get_pointer_safety`) |
| `__STDCPP_THREADS__`               | `1` if the program can have more than one thread             |

Particular implementations may define additional constants.