[TOC]

# Statements and flow control

But programs are not limited to a linear sequence of statements. During its process, a program may repeat segments of code, or take decisions and bifurcate. For that purpose, C++ provides flow control statements that serve to specify what has to be done by our program, when, and under which circumstances.

A compound statement is a group of statements (each of them terminated by its own semicolon), but all grouped together in a block, enclosed in curly braces `{}`.

## Selection statements

### if-else

```c++
if (x > 0)
  cout << "x is positive";
else if (x < 0)
  cout << "x is negative";
else
  cout << "x is 0";
```

### switch

```
switch (x) {
  case 1:
    cout << "x is 1";
    break;
  case 2:
    cout << "x is 2";
    break;
  default:
    cout << "value of x unknown";
  }
```

The `switch` statement has a somewhat peculiar syntax inherited from the early times of the first C compilers, because it uses labels instead of blocks. In the most typical use (shown above), this means that `break` statements are needed after each group of statements for a particular label. If `break` is not included, all statements following the case (including those under any other labels) are also executed, until the end of the switch block or a jump statement (such as `break`) is reached.

Notice that `switch` is limited to compare its evaluated expression against labels that are **constant expressions**. It is not possible to use variables as labels or ranges, because they are not valid C++ constant expressions.

## Iteration statements

Loops repeat a statement a certain number of times, or while a condition is fulfilled. They are introduced by the keywords `while`, `do`, and `for`.

### while

The while-loop:

```c++
int n = 10;

while (n>0) {
	cout << n << ", ";
	--n;
}
```

### do-while

The do-while-loop:

```c++
string str;

do {
    cout << "Enter text: ";
    getline (cin,str);
    cout << "You entered: " << str << '\n';
} while (str != "goodbye");
```

The do-while loop is usually preferred over a while-loop when the statements needs to be executed at least once, such as when the condition that is checked to end of the loop is determined within the loop statement itself.

### for

The for-loop is designed to iterate a number of times.   Its syntax is:

```
for (initialization; condition; increase) statement;
```

The for loop provides specific locations to contain an `initialization` and an `increase` expression, executed before the loop begins the first time, and after each iteration, respectively. 

The three fields in a for-loop are optional. They can be left empty, but in all cases the semicolon signs between them are required.

The for-loop has another syntax, which is used exclusively with ranges:

```
for ( declaration : range ) statement;
```

This kind of for loop iterates over all the elements in `range`, where `declaration` declares some variable able to take the value of an element in this range. Ranges are sequences of elements, including arrays, containers, and any other type supporting the functions begin and end;  For example:

```c++
string str {"Hello!"};
for (char c : str)
{
	cout << "[" << c << "]";
}
```

Range based loops usually also make use of type deduction for the type of the elements with `auto`. Typically, the range-based loop above can also be written as:

```c++
for (auto c : str)
{
	cout << "[" << c << "]";
}
```

## Jump statements

Jump statements allow altering the flow of a program by performing jumps to specific locations.

### break

`break` leaves a loop, even if the condition for its end is not fulfilled. It can be used to end an infinite loop, or to force it to end before its natural end.

### continue

The `continue` statement causes the program to skip the rest of the loop in the current iteration, as if the end of the statement block had been reached, causing it to jump to the start of the following iteration.

### goto

`goto` allows to make an absolute jump to another point in the program. This unconditional jump ignores nesting levels, and does not cause any automatic stack unwinding. Therefore, it is a feature to use with care, and preferably within the same block of statements, especially in the presence of local variables.

# Functions

Functions allow to structure programs in segments of code to perform individual tasks.

In C++, a function is a group of statements that is given a name, and which can be called from some point of the program. The most common syntax to define a function is:  

```
type name ( parameter1, parameter2, ...) { statements }
```

## The process of function calling

Let's have a look at an example:

```c++
#include <iostream>
using namespace std;

int addition (int a, int b)
{
  int r;
  r=a+b;
  return r;
}

int main ()
{
  int z;
  z = addition (5,3);
  cout << "The result is " << z;
}
```

In the example above, `main` calls `addition` function. The call to a function follows a structure very similar to its declaration. 

At the point at which the function `addition` is called from within `main`, the control is passed to function `addition`: here, execution of `main` is stopped, and will only resume once the `addition` function ends. At the moment of the function call, the value of both arguments (`5` and `3`) are copied to the local variables `int a` and `int b` within the `addition` function. Then run the code inside the `addition` function. 

Ends function `addition`, and returns the control back to the point where the function was called; in this case: to function `main`. At this precise moment, the program resumes its course on `main` returning exactly at the same point at which it was interrupted by the call to `addition`. But additionally, because `addition` has a return type, the call is evaluated as having a value, and this value is the value specified in the return statement that ended `addition`.

Therefore, the call to `addition` is an expression with the value returned by the function, and in this case, that value, 8, is assigned to `z`.

## Declaring and defining functions

In C++, identifiers can only be used in expressions once they have been declared. This rule applies to variables, functions, classes and so on.

Functions cannot be called before they are declared. That is why, in all the previous examples of functions, the functions were always defined before the `main` function, which is the function from where the other functions were called. If `main` were defined before the other functions, this would break the rule that functions shall be declared before being used, and thus would not compile.

### Separate declaring and defining

The prototype of a function can be declared without actually defining the function completely, giving just enough details to allow the types involved in a function call to be known. Naturally, the function shall be defined somewhere else, like later in the code. But at least, once declared like this, it can already be called.

The declaration shall include all types involved (the return type and the type of its arguments), using the same syntax as used in the definition of the function, but replacing the body of the function (the block of statements) with an ending semicolon.

The parameter list does not need to include the parameter names, but only their types. Parameter names can nevertheless be specified, but they are optional, and do not need to necessarily match those in the function definition. 

```c++
// declaring
int add(int x, int y);

// we can call the add function after it is declared

// defining
int add(int x, int y) {
	return x + y;
}
```

Declare the prototype of the functions. They already contain all what is necessary to call them, their name, the types of their argument, and their return type. With these prototype declarations in place, they can be called before they are entirely defined.

**Note:** If two functions are mutually called. We must declare them before call them.

## The ways of arguments passed

### Passed by value

In the functions seen earlier, arguments have always been passed *by value*. This means that, when calling a function, what is passed to the function are the values of these arguments on the moment of the call, which are copied into the variables represented by the function parameters. Any modification of these variables within the function has no effect on the values of the passed arguments variables outside it.

### Passed by reference

In certain cases, though, it may be useful to access an external variable from within a function. To do that, arguments can be passed *by reference*, instead of *by value*.

```c++
void duplicate (int& a, int& b, int& c)
{
  a*=2;
  b*=2;
  c*=2;
}
```

To gain access to its arguments, the function declares its parameters as *references*. In C++, references are indicated with an ampersand (`&`) following the parameter type.

When a variable is passed *by reference*, what is passed is no longer a copy, but the variable itself, the variable identified by the function parameter, becomes somehow associated with the argument passed to the function, and any modification on their corresponding local variables within the function are reflected in the variables passed as arguments in the call.

### Efficiency considerations and const references

#### Passed by reference is more efficiency

Calling a function with parameters taken by value causes copies of the values to be made. This is a relatively inexpensive operation for **fundamental types** such as `int`, but if the parameter is of a large **compound type**, it may result on certain overhead. Thus the way of passed by value is inefficiency.

We can avoid the copy operation via passed by reference. Arguments by reference do not require a copy. The function operates directly on (aliases of) the passed arguments, and, at most, it might mean the transfer of certain pointers to the function.

 #### Const reference to avoid be modified

On the flip side, functions with reference parameters are generally perceived as functions that modify the arguments passed, because that is why reference parameters are actually for.

The solution is for the function to guarantee that its reference parameters are not going to be modified by this function. This can be done by qualifying the parameters as constant:  

```c++
string concatenate (const string& a, const string& b)
{
  return a+b;
}
```

By qualifying them as `const`, the function is forbidden to modify the values of neither `a` nor `b`, but can actually access their values as references (aliases of the arguments), without having to make actual copies of the strings.

Therefore, `const` references provide functionality similar to passing arguments by value, but with an increased efficiency for parameters of large types. That is why they are extremely popular in C++ for arguments of compound types. Note though, that for most fundamental types, there is no noticeable difference in efficiency, and in some cases, const references may even be less efficient!

## Default values in parameters

In C++, functions can also have optional parameters, for which no arguments are required in the call, in such a way that, for example, a function with two parameters may be called with only one. For this, the function shall include a default value for its last parameter, which is used by the function when called with fewer arguments:

```c++
// function definition
int divide (int a, int b=2)
{
  int r;
  r=a/b;
  return (r);
}

// function call
divide(3);  // equal to divide(3, 2)
```

## Inline functions

Calling a function generally causes a certain overhead (stacking arguments, jumps, etc...), and thus for **very short functions**, it may be more efficient to simply insert the code of the function where it is called, instead of performing the process of formally calling a function.

Preceding a function declaration with the `inline` specifier informs the compiler that inline expansion is preferred over the usual function call mechanism for a specific function. This does not change at all the behavior of a function, but is merely used to suggest the compiler that the code generated by the function body shall be inserted at each point the function is called, instead of being invoked with a regular function call.  It's a mechanism for compiled-time.

For example:

```c++
inline string concatenate (const string& a, const string& b)
{
  return a+b;
}
```

**Note:** the keyword `inline` is only specified in the function declaration, not when it is called.

Most compilers already optimize code to generate inline functions when they see an opportunity to improve efficiency, even if not explicitly marked with the `inline` specifier. Therefore, this specifier merely indicates the compiler that inline is preferred for this function, although the compiler is free to not inline it, and optimize otherwise. In C++, optimization is a task delegated to the compiler, which is free to generate any code for as long as the resulting behavior is the one specified by the code.

## Functions with no type

The syntax requires the declaration to begin with a type. This is the type of the value returned by the function. But what if the function does not need to return a value? In this case, the type to be used is `void`, which is a special type to represent the absence of value.

And `void` can also be used in the function's parameter list to explicitly specify that the function takes no actual parameters when called. 

```c++
void printmessage (void)
{
  cout << "I'm a function!";
}
```

In C++, an empty parameter list can be used instead of `void` with same meaning, but the use of `void` in the argument list was popularized by the C language, where this is a requirement.

## The return of main function

You may have noticed that the return type of `main` is `int`, but most examples in this and earlier chapters did not actually return any value from `main`.

Well, there is a catch: If the execution of `main` ends normally without encountering a `return` statement the compiler assumes the function ends with an implicit return statement:  `return 0;`.

Note that this only applies to function `main` for historical reasons. All other functions with a return type shall end with a proper `return` statement that includes a return value, even if this is never used.

When `main` returns zero (either implicitly or explicitly), it is interpreted by the environment as that the program ended successfully. Other values may be returned by `main`, and some environments give access to that value to the caller in some way, although this behavior is not required nor necessarily portable between platforms. The values for `main` that are guaranteed to be interpreted in the same way on all platforms are:

| value          | description                                                  |
| -------------- | ------------------------------------------------------------ |
| `0`            | The program was successful                                   |
| `EXIT_SUCCESS` | The program was successful (same as above). This value is defined in header `<cstdlib>`. |
| `EXIT_FAILURE` | The program failed. This value is defined in header `<cstdlib>`. |

Because the implicit `return 0;` statement for `main` is a tricky exception, some authors consider it good practice to explicitly write the statement.

# Overloads and templates

## Function overloads

In C++, two different functions can have the same name if their parameters are different; either because they have a different number of parameters, or because any of their parameters are of a different type. Two functions with the same name are generally expected to have -at least- a similar behavior. For example:

```c++
int add(int x, int y) {
	return x + y;
}

double add(double x, double y) {
	return x + y;
}
```

**Note:** that a function cannot be overloaded only by its return type. At least one of its parameters must have a different type.

## Function templates

Overloaded functions may have the same definition, as the above example: `add` is overloaded with different parameter types, but with the exact same body. The function `add` could be overloaded for a lot of types, and it could make sense for all of them to have the same body. For cases such as this, C++ has the ability to define functions with **generic types**, known as **function templates**. 

### Templates defining

Defining a function template follows the same syntax as a regular function, except that it is preceded by the `template` keyword and a series of template parameters enclosed in angle-brackets `<>`:

```
template <template-parameters> function-declaration
```

The template parameters are a series of parameters separated by commas. These parameters can be generic template types by specifying either the `class` or `typename` keyword followed by an identifier. This identifier can then be used in the function declaration as if it was a regular type.  It makes no difference whether the generic type is specified with keyword `class` or keyword `typename` in the template argument list (they are 100% synonyms in template declarations).

For example:

```c++
// declare T as a generic type
template <class T>
T add(T x, T y) {
    T result;
    result = x + y;
	return result;
}
```

The above example declares `T` as a generic type, allows it be used anywhere in the function definition, just as any other type; it can be used as the type for parameters, as return type, or to declare new variables of this type. In all cases, it represents a generic type that will be determined on the moment the template is instantiated.

### Templates instantiation

Instantiating a template is applying the template to create a function using particular types or values for its template parameters. This is done by calling the *function template*, with the same syntax as calling a regular function, but specifying the template arguments enclosed in angle brackets:

```
name <template-arguments> (function-arguments)
```

For example, we can call the `add` template as follows:

```
add<int>(2, 3);
add<double>(2.1, 3.4);
```

In the example above, we used the function template `sum` twice. The first time with arguments of type `int`, and the second one with arguments of type `double`. The compiler has instantiated and then called each time the appropriate version of the function.

The compiler is even able to deduce the data type automatically without having to explicitly specify it within angle brackets if there is no unambiguous for types:

```
add(2, 3); // equal to add<int>(2, 3);
```

### Non-type template arguments

The template parameters can not only include types introduced by `class` or `typename`, but can also include expressions of a particular type, e.g. `int` as follows:

```c++
template <class T, int N>
T fixed_multiply (T val)
{
  return val * N;
}
```

The way of instantiation the template as follows:

```
fixed_multiply<int, 2>(23);
fixed_multiply<int, 5>(2);
```

Note: the value of template parameters is determined on **compile-time** to generate a different instantiation of the function `fixed_multiply`, and thus the value of that argument is never passed during runtime: The two calls to `fixed_multiply` in essentially call two versions of the function: one that always multiplies by two, and one that always multiplies by three. For that same reason, the second template argument needs to be a constant expression (it cannot be passed a variable).

# Name visibility

## Scopes

Named entities, such as variables, functions, and compound types need to be declared before being used in C++. The point in the program where this declaration happens influences its visibility:

* An entity declared outside any block has **global scope**, meaning that its name is valid anywhere in the code.
* While an entity declared within a block, such as a function or a selective statement, has **block scope**, and is only visible within the specific block in which it is declared, but not outside it. Variables with block scope are known as *local variables*.

In each scope, a name can only represent one entity. For example, there cannot be two variables with the same name in the same scope.

The visibility of an entity with *block scope* extends until the end of the block, including inner blocks. Nevertheless, an inner block, because it is a different block, can re-utilize a name existing in an outer scope to refer to a different entity; in this case, the name will refer to a different entity only within the inner block, hiding the entity it names outside. 

Variables declared in declarations that introduce a block, such as function parameters and variables declared in loops and conditions (such as those declared on a for or an if) are local to the block they introduce.

## Namespaces

Only one entity can exist with a particular name in a particular scope. This is seldom a problem for local names, since blocks tend to be relatively short. But **non-local names** (global scope) bring more possibilities for **name collision**, especially considering that libraries may declare many functions, types, and variables, neither of them local in nature, and some of them very generic.

Namespaces allow us to group named entities that otherwise would have **global scope** into narrower scopes, giving them **namespace scope** (logical scope). This allows organizing the elements of programs into different logical scopes referred to by names. Namespaces are particularly useful to avoid name collisions.

### Declaring namespaces

The syntax to declare a namespaces is:

```
namespace identifier
{
  named_entities declaring
}
```

Where `identifier` is any valid identifier and `named_entities` is the set of variables, types and functions that are included within the namespace. For example:

```c++
namespace Foo {
    int a, b;
}
```

These variables can be accessed from within their namespace normally, with their identifier (either `a` or `b`), but if accessed from outside the namespace they have to be properly qualified with the scope operator `::`. For example:

```
Foo::a;
Foo::b;
```

Another example:

```c++
#include <iostream>
using namespace std;

namespace foo
{
  int value() { return 5; }
}

namespace bar
{
  const double pi = 3.1416;
  double value() { return 2*pi; }
}

int main () {
  cout << foo::value() << '\n';
  cout << bar::value() << '\n';
  cout << bar::pi << '\n';
  return 0;
}
```

Namespaces can be split: Two segments of a code can be declared in the same namespace, for example:

```c++
namespace Foo {
    int a;
}
// something else
namespace Foo {
    int b;
}
```

### Using namespaces

The keyword `using` introduces a name into the current declarative region (such as a block), thus avoiding the need to qualify the name. For example:

```c++
#include <iostream>
using namespace std;

namespace first
{
  int x = 5;
  int y = 10;
}

namespace second
{
  double x = 3.1416;
  double y = 2.7183;
}

int main () {
  // introduce names into the current block
  using first::x;
  using second::y;
  
  cout << x << '\n';  // first::x
  cout << y << '\n';  // second::y
  cout << second::x << '\n';  // second::x
  cout << first::y << '\n';  // first::y
  return 0;
}
```

The keyword `using` can also be used as a directive to introduce an entire namespace:

```c++
int main () {
  // introduce an entire namespace into current block
  using namespace first;
  cout << x << '\n';  // first::x
  cout << y << '\n';  // first::y
  cout << second::x << '\n';
  cout << second::y << '\n';
  return 0;
}
```

`using` and `using namespace` have validity only in the same block in which they are stated or in the entire source code file if they are used directly in the global scope:

```c++
int main () {
  {
    using namespace first;
    cout << x << '\n';
  }
  {
    using namespace second;
    cout << x << '\n';
  }
  return 0;
}
```

### Namespace aliasing

Existing namespaces can be aliased with new names, with the following syntax:

```c++
namespace alias_name = current_name;
```

### The std namespace

All the entities (variables, types, constants, and functions) of the standard C++ library are declared within the `std`namespace. 

We add the `using namespace std;` in the global scope in there tutorials. This introduces direct visibility of all the names of the `std` namespace into the code. This is done in these tutorials to facilitate comprehension and shorten the length of the examples, but many programmers prefer to qualify each of the elements of the standard library used in their programs. 

## Storage lifetime

The storage for variables with *global* or *namespace scope* is allocated for the entire duration of the program. This is known as **static storage**, and it contrasts with the storage for *local variables* (those declared within a block). These use what is known as **automatic storage**. The storage for local variables is only available during the block in which they are declared; after that, that same storage may be used for a local variable of some other function, or used otherwise.

But there is another substantial difference between variables with *static storage* and variables with *automatic storage*:
- Variables with *static storage* (such as global variables) that are not explicitly initialized are automatically initialized to **zeroes**.
- Variables with *automatic storage* (such as local variables) that are not explicitly initialized are left uninitialized, and thus have an **undetermined value**.

