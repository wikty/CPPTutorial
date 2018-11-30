[TOC]



# Arrays

An array is a series of elements of the **same type** placed in **contiguous memory locations** that can be individually referenced by adding an **index** to a unique identifier.

## Declaring

Like a regular variable, an array must be declared before it is used. A typical declaration for an array in C++ is:

```
type name [elements];
```

where `type` is a valid type (such as `int`, `float`...), `name` is a valid identifier and the `elements` field (which is always enclosed in square brackets `[]`), specifies the length of the array in terms of the number of elements.

**Note:** The `elements` field within square brackets `[]`, representing the number of elements in the array, must be a **constant expression**, since arrays are blocks of static memory whose size must be determined at compile time, before the program runs.

## Initializing

Static arrays, and those declared directly in a namespace (outside any function), are always initialized. If no explicit initializer is specified, all the elements are default-initialized (with zeroes, for fundamental types).

But the arrays of *local scope* (for example, those declared within a function) are left uninitialized. This means that none of its elements are set to any particular value; their contents are undetermined at the point the array is declared.

But the elements in an array can be explicitly initialized to specific values when it is declared, by enclosing those initial values in braces `{}`. For example:

```
int foo [5] = { 16, 2, 77, 40, 12071 };
```

If declared with less values, the remaining elements are set to their default values (which for fundamental types, means they are filled with zero). For example:

```
int foo [5] = { 16, 2, 77 };  // the remaining elements foo[3] and foo[4] are zero
```

When an initialization of values is provided for an array, C++ allows the possibility of leaving the square brackets empty `[]`. In this case, the compiler will assume automatically a size for the array that matches the number of values included between the braces `{}`:

```
int foo[] = { 16, 2, 77, 40, 12071 };  // compiler automatically deduce the size
```

Finally, the evolution of C++ has led to the adoption of **universal initialization** also for arrays. Therefore, there is no longer need for the equal sign between the declaration and the initializer. Both these statements are equivalent:

```
int foo[] = { 10, 20, 30 };
int foo[] { 10, 20, 30 };  // universal initialization
```

## Indexing

The values of any of the elements in an array can be accessed just like the value of a regular variable of the same type. The syntax is:

```
name[index];
```

where `name` is the identifier of array, `index` is the index of the value we want to access. And in C++, the first element in an array is always numbered with a zero (not a one), no matter its length.

We can read and write from/to array by index syntax:

```c++
int foo[] = { 10, 20, 30 };
foo[0] = 75;  // write
int i = foo[0];  // read
```

**Note:** In C++, it is syntactically correct to exceed the valid range of indices for an array. This can create problems, since accessing out-of-range elements do not cause errors on compilation, but can cause errors on runtime. The reason for this being allowed will be seen in a later chapter when pointers are introduced.

## Multidimensional arrays

Multidimensional arrays can be described as "arrays of arrays". For example:

```c++
int map[20][30];
```

We can index multidimensional arrays as follows:

```c++
map[0][1]; // access the element of array in the first row and second column
```

Multidimensional arrays are not limited to two indices (i.e., two dimensions). They can contain as many indices as needed. Although be careful: the amount of memory needed for an array increases exponentially with each dimension. 

At the end, multidimensional arrays are just an abstraction for programmers, since the same results can be achieved with a simple array, by multiplying its indices:

```
int bar [3][5];   // is equivalent to
int bar [15];     // (3 * 5 = 15)  
```

With the only difference that with multidimensional arrays, the compiler automatically remembers the depth of each imaginary dimension. So you can access elements by index syntax. If you use simple arrays instead of multidimensional arrays, you must deal with the logical of access elements by yourself.

## Arrays as parameters

At some point, we may need to pass an array to a function as a parameter. In C++, it is not possible to pass the entire block of memory represented by an array to a function directly as an argument. But what can be **passed instead is its address**. In practice, this has almost the same effect, and it is a much faster and more efficient operation.

To accept an array as parameter for a function, the parameters can be declared as the array type, but with empty brackets, omitting the actual size of the array. For example:

```
void foo(int bar[]);
```

It would be enough to write a call like this:

```
int a[20];
foo(a);
```

But we have no idea of the size of the passed array inside the called function. How can we avoid to exceed the valid range of indices for the array? We can include a second parameter that tells the function the length of each array that we pass to it as its first parameter:

```
void foo(int bar[], int length);
```

But if we want to pass a multidimensional array to function `foo`, how can we determine the depth of each additional dimension? We can include other arguments to tell the function the depth of each dimension:

```
void foo(int bar[], int depth0, int depth1, int depth2);
```

Now we can pass a 3-dimension array to the function:

```
int a[3][4][5];

foo(a, 3, 4, 5);
```

Or in a function declaration, it is also possible to include multidimensional arrays:

```
void foo(int bar[][4][5]);
```

Notice that the first brackets `[]` are left empty, while the following ones specify sizes for their respective dimensions. This is necessary in order for the compiler to be able to determine the depth of each additional dimension.

In a way, passing an array as argument always loses a dimension. The reason behind is that, for historical reasons, arrays cannot be directly copied, and thus what is really passed is a pointer.

## Library arrays

The arrays explained above are directly implemented as a language feature, inherited from the C language. They are a great feature, but by restricting its copy and easily decay into pointers, they probably suffer from an excess of optimization.

To overcome some of these issues with language built-in arrays, C++ provides an alternative array type as a **standard container**. It is a **type template** (a class template, in fact) defined in header `<array>`.

Containers are a library feature that falls out of the scope of this tutorial, and thus the class will not be explained in detail here. Suffice it to say that they operate in a similar way to built-in arrays, except that they allow being copied (an actually expensive operation that copies the entire block of memory, and thus to use with care) and decay into pointers only when explicitly told to do so (by means of its member `data`).  

For example:

```c++
#include <iostream>
#include <array>
using namespace std;

int main()
{
  array<int,3> myarray {10,20,30};

  for (int i=0; i<myarray.size(); ++i)
    ++myarray[i];

  for (int elem : myarray)
    cout << elem << '\n';
}
```

# Character sequences

The `string` class has been briefly introduced in an earlier chapter. It is a very powerful class to handle and manipulate strings of characters. However, because strings are, in fact, sequences of characters, we can represent them also as plain arrays of elements of a `char` type. 

## Declaring

For example:

```
char str[20];
```

is an array that can store up to 20 elements of type `char`. Therefore, this array has a capacity to store sequences of up to 20 characters. But this capacity does not need to be fully exhausted: the array can also accommodate shorter sequences. For example, at some point in a program, either the sequence `"Hello"` or the sequence `"Merry Christmas"` can be stored in `foo`, since both would fit in a sequence with a capacity for 20 characters.

By convention, the end of strings represented in character sequences is signaled by a special character: the **null character**, whose literal value can be written as `'\0'`. Thus after the content of the string itself, a null character (`'\0'`) has been stored in order to indicate the end of the sequence.

 ## Initialization character sequences

Because arrays of characters are ordinary arrays, they follow the same rules as these. For example:

```
char s1[20] = {'b', 'a', 'r', '\0'};
char s2[] = {'b', 'a', 'r', '\0'};
char s3[] {'b', 'a', 'r', '\0'};
```

But arrays of character elements have another way to be initialized: using **string literals** (C-strings) directly. For example:

```
char s[] = "bar";  // char s[] = {'b', 'a', 'r', '\0'};
```

Sequences of characters enclosed in double-quotes (`"`) are *literal constants*. And their type is, in fact, a null-terminated array of characters. This means that string literals always have a null character (`'\0'`) automatically appended at the end.

**Note:** Here we are talking about initializing an array of characters at the moment it is being declared, and not about assigning values to them later (once they have already been declared). In fact, because string literals are regular arrays, they have the same restrictions as these, and cannot be assigned values.

The following expressions are invalid:

```
// assignment operation is invalid for char array
s = "hello";
s[] = "hello";
s = {'h', 'e', 'l', 'l', 'o', '\0'};
```

## Initialization strings

Plain arrays with null-terminated sequences of characters are the typical types used in the C language to represent strings (that is why they are also known as *C-strings*). 

In C++, even though the standard library defines a specific type for strings (class `string`), still, plain arrays with null-terminated sequences of characters (C-strings) are a natural way of representing strings in the language; in fact, string literals still always produce null-terminated character sequences, and not `string` objects.

  In the standard library, both representations for strings (C-strings and library strings) coexist, and most functions requiring strings are overloaded to support both.

For example, `cin` and `cout` support null-terminated sequences directly, allowing them to be directly extracted from `cin`or inserted into `cout`, just like strings. For example:  

```
char s1[] = "hello world!";
string s2 = "hell world!";

cout << s1 << s2;
```

**Note:** Character arrays have a fixed size that needs to be specified either implicit or explicitly when declared; while strings are simply strings, no size is specified. This is due to the fact that strings have a dynamic size determined during **runtime**, while the size of arrays is determined on **compilation**, before the program runs.

In any case, null-terminated character sequences and strings are easily transformed from one another. Null-terminated character sequences can be transformed into strings implicitly, and strings can be transformed into null-terminated character sequences by using either of `string`'s member functions `c_str` or `data`:  

```
char myntcs[] = "some text";
string mystring = myntcs;  // convert c-string to string
cout << mystring;          // printed as a library string
cout << mystring.c_str();  // printed as a c-string 
cout << mystring.data;     // printed as a c-string
```

# Pointers

In earlier chapters, **variables** have been explained as locations in the computer's memory which can be accessed by their **identifier** (their name). This way, the program does not need to care about the **physical address** of the data in memory; it simply uses the identifier whenever it needs to refer to the variable.

For a C++ program, *the memory of a computer is like a succession of memory cells, each one byte in size, and each with a unique address*. These single-byte memory cells are ordered in a way that allows data representations larger than one byte to occupy memory cells that have consecutive addresses.

This way, *each cell can be easily located in the memory by means of its unique address*. For example, the memory cell with the address `1776` always follows immediately after the cell with address `1775` and precedes the one with `1777`, and is exactly one thousand cells after `776` and exactly one thousand cells before `2776`.

When a variable is declared, the memory needed to store its value is assigned a specific location in memory (its memory address). Generally, C++ programs do not actively decide the exact memory addresses where its variables are stored. Fortunately, that task is left to the environment where the program is run - generally, *an operating system that decides the particular memory locations on runtime*. Thus the actual address of a variable in memory cannot be known before runtime.

However, it may be useful for a program to be able to obtain the address of a variable during runtime in order to access data cells that are at a certain position relative to it.  

## Address-of operator

The address of a variable can be obtained by preceding the name of a variable with an ampersand sign (`&`), known as *address-of operator*. For example: 

```
int bar = 5;
foo = &bar;  // assign the address of variable bar to foo
```

This would assign the address of variable `bar` to `foo`; by preceding the name of the variable `bar` with the *address-of operator* (`&`), we are no longer assigning the content `5` of the variable itself to `foo`, but its address (will be determined in runtime).

The variable that stores the address of another variable (like `foo` in the previous example) is what in C++ is called a **pointer**. Pointers are a very powerful feature of the language that has many uses in lower level programming.

## Dereference operator

As just seen, a variable which stores the address of another variable is called a *pointer*. Pointers are said to "point to" the variable whose address they store.

An interesting property of pointers is that they can be used to access the variable they point to directly. This is done by preceding the pointer name with the *dereference operator* (`*`). The operator itself can be read as "value pointed to by".  For example:

```
baz = *foo;  // baz equal to value pointed to by foo
```

The reference and dereference operators are thus complementary:

- `&` is the **address-of operator**, and can be read simply as **"address of"**
- `*` is the **dereference operator**, and can be read as **"value pointed to by"**

Thus, they have sort of opposite meanings: An address obtained with `&` can be dereferenced with `*`.

## Declaring pointers

Due to the ability of a pointer to directly refer to the value that it points to, a pointer has different properties when it points to a `char` than when it points to an `int` or a `float`. Once dereferenced, the type needs to be known. And for that, the declaration of a pointer needs to include the data type the pointer is going to point to. The syntax of declaring pointer as follows:

```
type * name;
```

where `type` is the data type pointed to by the pointer. This type is not the type of the pointer itself, but the type of the data the pointer points to.

Note that the asterisk (`*`) used when declaring a pointer only means that it is a pointer (it is part of its type compound specifier), and should not be confused with the *dereference operator* seen a bit earlier, but which is also written with an asterisk (`*`). They are simply two different things represented with the same sign.

Now you may wonder what is the data type of pointer self? Pointer variables store memory address of other variables, and the address value is an integer, so you can think the data type of pointer self is a kind of integer. And the size in memory of a pointer depends on the platform where the program runs.

A pointer may point to different variables during its lifetime in a program. For example:

```c++
int main ()
{
  int firstvalue = 5, secondvalue = 15;
  int * p1, * p2;

  p1 = &firstvalue;  // p1 = address of firstvalue
  p2 = &secondvalue; // p2 = address of secondvalue
  *p1 = 10;          // value pointed to by p1 = 10
  *p2 = *p1;         // value pointed to by p2 = value pointed to by p1
  p1 = p2;           // p1 = p2 (value of pointer is copied)
  *p1 = 20;          // value pointed to by p1 = 20
  
  cout << "firstvalue is " << firstvalue << '\n';
  cout << "secondvalue is " << secondvalue << '\n';
  return 0;
}
```

## Initialization pointers

Pointers can be initialized either to the address of a variable, or to the value of another pointer (or array):

```c++
int v;
int *foo = &v;
int *bar = foo;
```

## Pointer arithmetics

To conduct arithmetical operations on pointers is a little different than to conduct them on regular integer types. To begin with, only **addition** and **subtraction** operations are allowed; the others make no sense in the world of pointers. But both addition and subtraction have a slightly different behavior with pointers, according to the size of the data type to which they point.

When fundamental data types were introduced, we saw that types have different sizes. For example: `char` always has a size of 1 byte, `short` is generally larger than that, and `int` and `long` are even larger; the exact size of these being dependent on the system. For example, let's imagine that in a given system, `char` takes 1 byte, `short` takes 2 bytes, and `long` takes 4.  We have three pointers for those types:

```
char *pc; // assume it points memory locations 1000
short *ps;  // assume it points memory locations 2000
long *pl;  // assume it points memory locations 3000
```

If we add one to those pointer, how can them changed:

```
++pc;  // points to 1001
++ps;  // points to 2002
++pl;  // points to 3004
```

The reason is that, when adding one to a pointer, the pointer is made to point to the following element of the same type, and, therefore, the size in bytes of the type it points to is added to the pointer.

This also applies to expressions incrementing and decrementing pointers, which can become part of more complicated expressions that also include dereference operators (`*`). Remembering operator precedence rules, we can recall that postfix operators, such as increment and decrement, have higher precedence than prefix operators, such as the dereference operator (`*`). Therefore, the following expression:

```
*p++;  // equal to: *(p++), i.e. the originally pointed value will be returned
```

There are other expressions may be confused:

```
*p++   // same as *(p++): increment pointer, and dereference unincremented address
*++p   // same as *(++p): increment pointer, and dereference incremented address
++*p   // same as ++(*p): dereference pointer, and increment the value it points to
(*p)++ // dereference pointer, and post-increment the value it points to
```

## Pointers and arrays

The concept of arrays is related to that of pointers. In fact, *arrays work very much like pointers to their first elements*, and, actually, an array can always be implicitly converted to the pointer of the proper type. For example:

```
int a[20];
int *p;
p = a;  // p points to the first element of array a
```

`a` and `p` would be equivalent and would have very similar properties. The main difference being that `p` can be assigned a different address, whereas `a` can never be assigned anything, and will always represent the same block of 20 elements of type `int`, i.e. `a = p` is invalid.

Access array by pointer, for example:

```c++
#include <iostream>
using namespace std;

int main ()
{
  int numbers[5];
  int * p;
  p = numbers;  	// p points to numbers[0]
  *p = 10;		
  p++;  			// p points to numbers[1]
  *p = 20;      
  p = &numbers[2];  // p points to numbers[2]
  *p = 30;      
  p = numbers + 3;  // p points to numbers[3]
  *p = 40;    
  p = numbers;  
  *(p+4) = 50;      // (p+4) points to numbers[4]
  
  for (int n=0; n<5; n++)
    cout << numbers[n] << ", ";
  
  return 0;
}
```

Pointers and arrays support the same set of operations, with the same meaning for both. The main difference being that pointers can be assigned new addresses, while arrays cannot.

In the chapter about arrays, brackets (`[]`) were explained as specifying the index of an element of the array. Well, in fact these brackets are a **dereferencing operator** known as **offset operator**. They dereference the variable they follow just as `*` does, but they also add the number between brackets to the address being dereferenced.  For example:

```c++
a[5];
*(a+5);
```

These two expressions are equivalent and valid, not only if `a` is a pointer, but also if `a` is an array. Remember that if an array, its name can be used just like a pointer to its first element.

## Pointers and string literals

String literals are arrays of the proper array type to contain all its characters plus the terminating null-character, with each of the elements being of type `const char` (as literals, they can never be modified). For example:

```
const char * foo = "hello";
```

This declares an array with the literal representation for `"hello"`, and then a pointer to its first element is assigned to `foo`. 

The pointer `foo` points to a sequence of characters. And because pointers and arrays behave essentially in the same way in expressions, `foo` can be used to access the characters in the same way arrays of null-terminated character sequences are. For example:

```
*(foo+4);
foo[4];
```

## Pointers and const

Pointers can be used to access a variable by its address, and this access may include modifying the value pointed. But it is also possible to declare pointers that can access the pointed value to read it, but not to modify it. For this, it is enough with qualifying the type pointed to by the pointer as `const`. 

### Declaring pointers pointed to constants

For example:

```c++
int x;
int y = 10;
const int* p = &y;
x = *p; // read const pointer allowed
// *p = x; // write const pointer NOT allowed
```

Here `p` points to a variable, but points to it in a `const`-qualified manner, meaning that it can read the value pointed, but it cannot modify it. Note also, that the expression `&y` is of type `int*`, but this is assigned to a pointer of type `const int*`. This is allowed: a pointer to non-const can be **implicitly converted** to a pointer to const. But not the other way around! As a safety feature, pointers to `const` are not implicitly convertible to pointers to non-`const`, i.e. `int *q = p;` is not allowed.

### Declaring pointers themselves are constants

And this is where a second dimension to constness is added to pointers: Pointers can also be themselves const. And this is specified by appending const to the pointed type (after the asterisk):

```c++
int x;
      int *       p1 = &x;  // non-const pointer to non-const int
const int *       p2 = &x;  // non-const pointer to const int
      int * const p3 = &x;  // const pointer to non-const int
const int * const p4 = &x;  // const pointer to const int
```

To add a little bit more confusion to the syntax of `const` with pointers, the `const` qualifier can either precede or follow the pointed type, with the exact same meaning:

```c++
const int * p2a = &x;  //      non-const pointer to const int
int const * p2b = &x;  // also non-const pointer to const int
```

In summary, the `const` qualifier precedes the pointer declaring identifier asterisk (`*`), then we cannot modify the value pointed by this pointer. The `const` qualifier follows the asterisk (`*`), then we cannot modify the pointer.

### Const parameters

One of the use cases of pointers to `const` elements is as function parameters: a function that takes a pointer to non-`const` as parameter can modify the value passed as argument, while a function that takes a pointer to `const` as parameter cannot.

```c++
void increment_all (int* start, int* stop)
{
  int * current = start;
  while (current != stop) {
    ++(*current);  // increment value pointed
    ++current;     // increment pointer
  }
}

void print_all (const int* start, const int* stop)
{
  // pointers point to constant content they cannot modify, but they are not constant themselves
  const int * current = start;
  while (current != stop) {
    cout << *current << '\n';
    ++current;     // increment pointer
  }
}

int main ()
{
  int numbers[] = {10,20,30};
  increment_all (numbers,numbers+3);
  print_all (numbers,numbers+3);
  return 0;
}
```

## Pointers to pointers

C++ allows the use of pointers that point to pointers, that these, in its turn, point to data (or even to other pointers). The syntax simply requires an asterisk (`*`) for each level of indirection in the declaration of the pointer:

```
int x = 4;
int* p = &x;
int** pp = &p;
```

## Pointers to functions

C++ allows operations with pointers to functions. The typical use of this is for passing a function as an argument to another function. Pointers to functions are declared with the same syntax as a regular function declaration, except that the name of the function is enclosed between parentheses () and an asterisk (`*`) is inserted before the name:

```c++
#include <iostream>
using namespace std;

int addition (int a, int b)
{ return (a+b); }

int subtraction (int a, int b)
{ return (a-b); }

int operation (int x, int y, int (*functocall)(int,int))
{
  int g;
  g = (*functocall)(x,y);
  return (g);
}

int main ()
{
  int m,n;
  int (*minus)(int,int) = subtraction;

  m = operation (7, 5, addition);
  n = operation (20, m, minus);
  cout <<n;
  return 0;
}
```

## Void pointers

The `void` type of pointer is a special type of pointer. In C++, `void` represents the absence of type. *Therefore, `void` pointers are pointers that point to a value that has no type (and thus also an undetermined length and undetermined dereferencing properties).*

This gives `void` pointers a great flexibility, by being able to point to any data type, from an integer value or a float to a string of characters. In exchange, they have a great limitation: *the data pointed to by them cannot be directly dereferenced* (which is logical, since we have no type to dereference to), and for that reason, any address in a `void`pointer needs to be transformed into some other pointer type that points to a concrete data type before being dereferenced.

One of its possible uses may be to pass *generic parameters* to a function. For example:   

```c++
#include <iostream>
using namespace std;

void increase (void* data, int psize)
{
  if ( psize == sizeof(char) )
  {
      char* pchar; 
      pchar=(char*)data; // convert void pointer to char pointer
      ++(*pchar); 
  }
  else if (psize == sizeof(int) )
  { 
      int* pint; 
      pint=(int*)data;  // convert void pointer to int pointer
      ++(*pint); 
  }
}

int main ()
{
  char a = 'x';
  int b = 1602;
  increase (&a,sizeof(a));
  increase (&b,sizeof(b));
  cout << a << ", " << b << '\n';
  return 0;
}
```

## Invalid pointers and null pointers

In principle, pointers are meant to point to valid addresses, such as the address of a variable or the address of an element in an array. But pointers can actually point to any address, including addresses that do not refer to any valid element. Typical examples of this are **uninitialized pointers** and **pointers to nonexistent elements of an array**:

```
int * p;               // invalid: uninitialized pointer

int myarray[10];
int * q = myarray+20;  // invalid: point to element out of bounds 
```

Neither `p` nor `q` point to addresses known to contain a value, but none of the above statements causes an error. In C++, pointers are allowed to take any address value, no matter whether there actually is something at that address or not. What can cause an error is to dereference such a pointer (i.e., actually accessing the value they point to). *Accessing such a pointer causes undefined behavior, ranging from an error during runtime to accessing some random value.*

But, sometimes, a pointer really needs to explicitly **point to nowhere**, and not just an invalid address. For such cases, there exists a special value that any pointer type can take: the **null pointer value**. This value can be expressed in C++ in two ways: either with an integer value of zero, or with the `nullptr` keyword:

```
int *p = 0;
int *q = nullptr;
```

It is also quite usual to see the defined constant `NULL` be used in older code to refer to the *null pointer* value:

```
int *p = NULL;
```

`NULL` is defined in several headers of the standard library, and is defined as an alias of some *null pointer* constant value (such as `0` or `nullptr`).

# Dynamic memory

In the programs seen in previous chapters, all memory needs were determined before program execution by defining the variables needed. But *there may be cases where the memory needs of a program can only be determined during runtime*. For example, when the memory needed depends on user input. On these cases, programs need to **dynamically allocate memory**, for which the C++ language integrates the operators `new` and `delete`.

## Allocate

Dynamic memory is allocated using operator `new`. `new` is followed by a data type specifier and, if a sequence of more than one element is required, the number of these within brackets `[]`. *It returns a pointer to the beginning of the new block of memory allocated*. 

### Syntax

Its syntax is: 

```
pointer = new type
pointer = new type [number_of_elements]
```

The first expression is used to allocate memory to contain one single element of type `type`. The second one is used to allocate a block (an array) of elements of type `type`, where `number_of_elements` is an integer value representing the amount of these.

### The differences with local array variables

There is a substantial difference between declaring a normal array and allocating dynamic memory for a block of memory using `new`. The most important difference is that the size of a regular array needs to be a **constant expression**, and thus its size has to be determined at the moment of designing the program, before it is run, whereas the dynamic memory allocation performed by `new` allows to assign memory during runtime using **any variable value as size**.

### Check allocation

The dynamic memory requested by our program is allocated by the system from the **memory heap**. However, computer memory is a limited resource, and it can be exhausted. Therefore, there are no guarantees that all requests to allocate memory using operator `new` are going to be granted by the system. 

C++ provides two standard mechanisms to check if the allocation was successful:  

* One is by handling exceptions. Using this method, an exception of type `bad_alloc` is thrown when the allocation fails. This exception method is the method used by default by `new`, and is the one used in a declaration like: `int *p = new int [5];`.
* The other method is known as `nothrow`, and what happens when it is used is that when a memory allocation fails, instead of throwing a `bad_alloc` exception or terminating the program, the pointer returned by `new` is a **null pointer**, and the program continues its execution normally. This method can be specified by using a special object called `nothrow`, declared in header `<new>`, as argument for `new`: `int *p = new (nothrow) int [5];`.

It is considered good practice for programs to always be able to handle failures to allocate memory, either by checking the pointer value (if `nothrow`) or by catching the proper exception.

The `nothrow` method is likely to produce less efficient code than exceptions, since it implies explicitly checking the pointer value returned after each and every allocation. Therefore, the exception mechanism is generally preferred, at least for critical allocations. Still, most of the coming examples will use the `nothrow` mechanism due to its simplicity.

## Free

In most cases, memory allocated dynamically is only needed during specific periods of time within a program; once it is no longer needed, it can be freed so that the memory becomes available again for other requests of dynamic memory. This is the purpose of operator `delete`, whose syntax is:

```
delete pointer;
delete[] pointer;
```

The first statement releases the memory of a single element allocated using `new`, and the second one releases the memory allocated for arrays of elements using new and a size in brackets (`[]`).

And the value `pointer` passed as argument to `delete` shall be either a pointer to a memory block previously allocated with `new`, or a *null pointer* (in the case of a *null pointer*, `delete` produces no effect).

## Dynamic memory in C

  C++ integrates the operators `new` and `delete` for allocating dynamic memory. But these were not available in the C language; instead, it used a library solution, with the functions `malloc`, `calloc`, `realloc` and `free`, defined in the header `<cstdlib>` (known as `<stdlib.h>` in C). The functions are also available in C++ and can also be used to allocate and deallocate dynamic memory.

Note, though, that the memory blocks allocated by these functions are not necessarily compatible with those returned by `new`, so they should not be mixed; each one should be handled with its own set of functions or operators.  

# Data Structures

A **data structure** is a group of data elements grouped together under one name. These data elements, known as **members**, can have different types and different lengths. 

## Declaring

Data structures can be declared in C++ using the following syntax:

```
struct type_name {
member_type1 member_name1;
member_type2 member_name2;
member_type3 member_name3;
} object_names;
```

where `type_name` is a name for the structure type, `object_name` (optional) can be a set of valid identifiers for objects (split by comma) that have the type of this structure. Within braces `{}`, there is a list with the data members, each one is specified with a type and a valid identifier as its name.

For example:

```
struct Point {
    double x;
    double y;
} p1, p2;

Point p3;
```

## Access members

Once the objects of a determined structure type are declared its members can be accessed directly. The syntax for that is simply to insert a dot (`.`) between the object name and the member name. For example:

```
p1.x = 1.0;
p2.y = 2.0;
p3.x = 3.0;
```

## Array of structures

```
struct Point {
    double x;
    double y;
} points[10];
```

The member of elements can be access by:

```
points[2].x = 2.3;
```

## Pass to function

```c++
void print(Point p) {
	std::cout << p.x << ", " << p.y;
}

Point m;
m.x = 2.3;
m.y = 3.2;
print(m);
```

## Pointers to structures

Like any other type, structures can be pointed to by its own type of pointers:

```
Point p;
Point* ptr = &p;
```

We can access the members of structure by `->` syntax:

```
ptr->x;  // equal to: (*ptr).x
ptr->y;  // equal to: (*ptr).y
```

The arrow operator (`->`) is a dereference operator that is used exclusively with pointers to objects that have members. This operator serves to access the member of an object directly from its address.

## Nesting structures

Structures can also be nested in such a way that an element of a structure is itself another structure:

```
struct Point {
    double x;
    double y;
};

struct Line {
	Point p1, p2;
	string name;
} l1;

Line *p = &l1;
p->p1.x;
p->p2.y;
p->name;
```

# Other data types

## Unions

Unions allow one portion of memory to be accessed as different data types. Its declaration and use is similar to the one of structures, but its functionality is totally different:

```
union type_name {
member_type1 member_name1;
member_type2 member_name2;
member_type3 member_name3;
} object_names;
```

This creates a new union type, identified by `type_name`, in which *all its member elements occupy the same physical space in memory*. The size of this type is the one of the largest member element. Since all of members are referring to the same location in memory, the modification of one of the members will affect the value of all of them. It is not possible to store different values in them in a way that each is independent of the others.

One of the uses of a union is to be able to access a value either in its entirety or as an array or structure of smaller elements. For example: 

```
union mix_t {
  int l;       // assume int is 4 bytes
  struct {
    short hi;  // assume short is 2 bytes
    short lo;  // assume short is 2 bytes
    } s;
  char c[4];
} mix;
```

If we assume that the system where this program runs has an `int` type with a size of 4 bytes, and a `short` type of 2 bytes, the union defined above allows the access to the same group of 4 bytes: `mix.l`, `mix.s` and `mix.c`, and which we can use according to how we want to access these bytes: as if they were a single value of type `int`, or as if they were two values of type `short`, or as an array of `char` elements, respectively. The example mixes types, arrays, and structures in the union to demonstrate different ways to access the data.

## Anonymous unions

When unions are members of a class (or structure), they can be declared with no name. In this case, they become *anonymous unions*, and its members are directly accessible from objects by their member names. For example, see the differences between these two structure declarations: 

```c++
// regular union
struct book1_t {
  char title[50];
  char author[50];
  union {
    float dollars;
    int yen;
  } price;
} book1;

book1.price.dollars;
book1.price.yen;

// anonymous union
struct book2_t {
  char title[50];
  char author[50];
  union {
    float dollars;
    int yen;
  };
} book2;

book2.dollars;
book2.yen;
```

## Enumerated types

Enumerated types are types that are defined with a set of custom identifiers, known as *enumerators*, as possible values. Objects of these *enumerated types* can take any of these enumerators as value:

```
enum type_name {
  value1,
  value2,
  value3
} object_names;
```

This creates the type `type_name`, which can take any of `value1`, `value2`, `value3`, ... as value. Objects (variables) of this type can directly be instantiated as `object_names`.

Values of *enumerated types* declared with `enum` are *implicitly convertible to an integer type, and vice versa*. In fact, the elements of such an `enum` are always assigned an integer numerical equivalent internally, to which they can be implicitly converted to or from. If it is not specified otherwise, the integer value equivalent to the first possible value is `0`, the equivalent to the second is `1`, to the third is `2`, and so on. A specific integer value can be specified for any of the possible values in the enumerated type. And if the constant value that follows it is itself not given its own value, it is automatically assumed to be the same value plus one.

Notice that enum declaration includes no other type, neither fundamental nor compound, in its definition. To say it another way, somehow, this creates a whole new data type from scratch without basing it on any other existing type. The possible values that variables of this new type may take are the enumerators listed within braces. For example:

```c++
enum colors_t {black, blue, green, cyan, red, purple, yellow, white};

colors_t c = black;
if (c == black) c = blue;
```

## Enumerated types with enum class

But, in C++, it is possible to create real `enum` types that are neither implicitly convertible to `int` and that neither have enumerator values of type `int`, but of the `enum` type itself, thus preserving type safety. They are declared with `enum class`(or `enum struct`) instead of just `enum`:

```c++
enum class Colors {black, blue, green, cyan, red, purple, yellow, white};

// Each of the enumerator values of an enum class type needs to be scoped into its type
Colors c = Colors::black;
```

Enumerated types declared with `enum class` also have more control over their underlying type; it may be any integral data type, such as `char`, `short` or `unsigned int`, which essentially serves to determine the size of the type. For example:

```c++
enum class EyeColor : char {blue, green, brown}; 
```

## Type aliases

A type alias is a different name by which a type can be identified. In C++, any valid type can be aliased so that it can be referred to with a different identifier.

In C++, there are two syntaxes for creating such type aliases: The first, inherited from the C language, uses the `typedef` keyword:

```c++
typedef existing_type new_type_name ;
```

More recently, a second syntax to define type aliases was introduced in the C++ language:

```c++
using new_type_name = existing_type ;
```

Both aliases defined with `typedef` and aliases defined with `using` are semantically equivalent. The only difference being that `typedef` has certain limitations in the realm of templates that `using` has not. Therefore, `using` is more generic, although `typedef` has a longer history and is probably more common in existing code. Note that neither `typedef` nor `using` create new distinct data types. They only create synonyms of existing types.  

Type aliases can be used to reduce the length of long or confusing type names, but they are most useful as tools to abstract programs from the underlying types they use. For example, by using an alias of `int` to refer to a particular kind of parameter instead of using `int` directly, it allows for the type to be easily replaced by `long` (or some other type) in a later version, without having to change every instance where it is used.