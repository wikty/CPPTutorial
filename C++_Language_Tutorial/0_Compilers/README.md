## Why we need a compiler?

Computers understand only one language and that language consists of sets of instructions made of ones and zeros. This computer language is appropriately called **machine language**.

As you can imagine, programming a computer directly in machine language using only ones and zeros is very tedious and error prone. To make programming easier, **high level languages** have been developed. High level programs also make it easier for programmers to inspect and understand each other's programs easier.

Because a computer can only understand machine language and humans wish to write in high level languages high level languages have to be re-written (translated) into machine language at some point. This is done by special programs called **compilers**, **interpreters**, or **assemblers** that are built into the various programming applications.

C++ is designed to be a compiled language, meaning that it is generally translated into machine language that can be understood directly by the system, making the generated program highly efficient. For that, a set of tools are needed, known as the development toolchain, whose core are a compiler and its linker.

## Which compiler we need?

C++ is a language that has evolved much over the years, and these tutorials explain many features added recently to the language. Therefore, in order to properly follow the tutorials, a recent compiler is needed. It shall support (even if only partially) the features introduced by the 2011 standard (**C++ 11**).

Additional, the examples in these tutorials are all **console programs**. Console programs are programs that use text to communicate with the user and the environment, such as printing text to the screen or reading input from a keyboard. Console programs are easy to interact with, and generally have a predictable behavior that is identical across all platforms.

The easiest way for beginners to compile C++ programs is by using an Integrated Development Environment (**IDE**). An IDE generally integrates several development tools, including a text editor and tools to compile programs directly from it.

There are some free IDEs:

| IDE                       | Platform            | Console programs                                             |
| ------------------------- | ------------------- | ------------------------------------------------------------ |
| **Code::blocks**          | Windows/Linux/MacOS | [Compile console programs using Code::blocks](http://www.cplusplus.com/doc/tutorial/introduction/codeblocks/) |
| **Visual Studio Express** | Windows             | [Compile console programs using VS Express 2013](http://www.cplusplus.com/doc/tutorial/introduction/visualstudio/) |
| **Dev-C++**               | Windows             | [Compile console programs using Dev-C++](http://www.cplusplus.com/doc/tutorial/introduction/devcpp/) |

If you compile your source code in the terminal, please remember to enable **C++ 11** flags:

* For GCC: `g++ -std=c++0x example.cpp -o example_program`
* For Clang: ``clang++ -std=c++11 -stdlib=libc++ example.cpp -o example_program`` 