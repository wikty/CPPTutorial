# Item 13: 用对象来管理资源

什么是资源，为什么要进行资源管理？

>  Resource Management is something that, once you’re done using it, you need to
> return to the system. If you don’t, bad things happen. In C++ programs,
> the most commonly used resource is dynamically allocated
> memory (if you allocate memory and never deallocate it, you’ve got a
> memory leak), but memory is only one of many resources you must
> manage. Other common resources include file descriptors, mutex
> locks, fonts and brushes in graphical user interfaces (GUIs), database
> connections, and network sockets. Regardless of the resource, it’s
> important that it be released when you’re finished with it.

## 手动管理

先来看一个手动资源管理的例子。我们有各种投资产品需要处理，如：股票、债券等，不同的投资产品由工厂函数来创建：

```c++
// 基类
class Investment { };

// 各种投资产品
class StockInvestment: public Investment {};

class BondInvestment: public Investment {};

// 工厂函数，通过动态分配内存来创建投资产品
Investment* createInvestment();
```

客户想要使用投资产品时，需要自己来管理内存资源的分配和释放：

```c++
void f() {
    Investment* p = createInvestment();
    
    // 使用资源对象...
    
    delete p;
}
```

手动管理资源的缺陷在于：客户不一定可以释放资源，可能客户忘记 `delete` 或者在使用资源的语句中提前 `return` 了或者使用期间引发了异常。更糟糕的是，代码后续的维护中，可能忘记需要释放资源而提前进行了 `return` 等处理。

## 内置的垃圾回收机制

对于许多动态分配于堆（heap）的资源，如：内存，可以利用管理对象自动回收资源，要点：

- 获得资源后立即放入管理对象中（Resource Acquisition Is Initialization, RAII）
- 利用管理对象的析构函数，确保资源一定被释放，即当控制流离开作用域区块时，资源应该被自动释放

### auto_ptr/unique_ptr

> This class template provides a limited *garbage collection* facility for pointers, by allowing pointers to have the elements they point to automatically destroyed when the *auto_ptr* object is itself destroyed.

客户可以这样使用该类来管理资源：

```c++
void f() {
    std::auto_ptr<Investment> p(createInvestment());  // 这里的 p 就如同 Investment* 一样
    
    // 使用资源对象...
    
    // auto_ptr 对象的析构函数会自动释放资源的
}
```

`auto_ptr` 对其所指向的资源对象具有唯一的所有权，也即没有两个 `auto_ptr` 对象会指向同一个资源对象，不同 `auto_ptr` 对象之间的复制会移交所有权并将交出方设为空指针。

> no two `auto_ptr` objects should *own* the same element, since both would try to destruct them at some point. When an assignment operation takes place between two `auto_ptr` objects, *ownership* is transferred, which means that the object losing ownership is set to no longer point to the element (it is set to the *null pointer*).

不过 `auto_ptr` 在 C++ 11 中被废弃了，取而代之的是 `unique_ptr`。

### shared_ptr

> Manages the storage of a pointer, providing a limited *garbage-collection* facility, possibly sharing that management with other objects.
>
> Objects of `shared_ptr` types have the ability of *taking ownership* of a pointer and *share* that ownership: once they take ownership, the group of owners of a pointer become responsible for its deletion when the last one of them releases that ownership.  

由 `shared_ptr` 实现的管理对象，支持引用计数，也即资源对象可以在多个管理对象之间共享，直到最后一个引用该资源对象的管理对象销毁时，资源对象被释放。

```c++
void f() {
    std::shared_ptr<Investment> p(createInvestment());
    std::shared_ptr<Investment> p1(p);		// p 和 p1 均指向同一个资源对象
}
```

值得注意的是 `auto_ptr` 和 `shared_ptr` 都只管理单个资源对象，不管理资源对象数组，也即释放资源时使用 `delete` 而非 `delete []`。可以使用 `std::vector` 来表示资源数组。

有时候，如果上述内置的垃圾回收机制无法支持你对资源管理的需求，可以考虑自定义资源管理类。

# Item 14: 资源管理类的复制行为

Item 13 演示了如何利用 `auto_ptr` 和 `shared_ptr` 来管理动态创建于 heap 上的资源，当资源对象不是基于 heap 时，可能就需要自己来定义资源管理类。

假设我们使用 C API 来处理锁对象的锁定和解锁：

```c++
void lock(Mutex* pm);
void unlock(Mutex* pm);
```

为了确保锁对象被解锁，可以通过自定义资源管理类来管理它，这里锁是资源，不过并不是要销毁它，而是对它解锁：

```c++
class Lock {
  private:
    Mutex* pm;
  public:
    // 创建管理对象时，锁定
    explicit Lock(Mutex *p): pm(p) {
        lock(pm);
    }
    // 管理对象销毁时，解锁
    ~lock() {
        unlock(pm);
    }
};
```

客户使用该 `Lock` 管理类的示例：

```c++
Mutex m;	// 创建互斥器

{// 创建 block 用来定义临界区
    
    Lock lk(&m);	// 锁定互斥器
    
    // ...			// 临界代码
    
} // 离开临界区，管理对象销毁，互斥器解除锁定
```

对于自定义资源管理类，有一个问题需要考虑：我们如何处理管理对象的复制，比如上述的 `Lock` 对象被复制时，我们该如何处理这个复制行为呢？有以下几个方案：

- 禁止复制。在大多数情况下，我们并不想要管理对象被复制，因为这可能会引起资源管理的混乱。如果不希望管理对象进行复制行为，可以使用 Item 6 介绍的方法来禁止管理对象复制。

- 转移资源的所有权。如果被管理的资源是独一无二的，那么管理对象之间的复制，其实意味着转移被管理资源对象的所有权。可以借助 `auto_ptr` 来实现所有权的转移。

- 管理对象复制等价于资源对象的引用计数。可以通过 `shared_ptr` 来改进 `Lock` 类使得它引用计数：

  ```c++
  class Lock {
    private:
      std::shared_ptr<Mutex> pm;
    public:
      explicit Lock(Mutex *p): pm(p, unlock) {
          lock(pm.get()); // get() 返回被管理的资源对象
      }
  };
  ```

- 复制底部资源对象。这是深度复制策略，不仅仅复制管理对象，还复制被管理的资源对象。

值得注意的是，如果想要实现自定义的复制行为，就需要实现 copy 构造和 copy 赋值成员函数，否则编译器会为它们创建默认函数（一般不符合你的需求）。

# Item 15: 在资源管理类中访问原始的资源对象

大多数情况下，我们直接使用管理对象即可，但有些 API 需要我们提供原始资源对象而非管理对象，此时我们需要直接访问资源对象，如下：

```c++
std::shared_ptr<Investment> pInv(createInvestment());

void print(const Investment *p); // 该函数不接受管理对象，仅接受资源对象
```

我们可以直接将 `shared_ptr` 对象传递给函数 `print` 吗？显然不行，由于类型不匹配，不能通过编译。为此我们需要直接访问资源对象，`auto_ptr` 和 `shared_ptr` 都提供了 `get()` 方法来实现**显式转换**为资源对象。

```c++
print(pInv.get());	// get() 返回指向资源对象的指针
```

此外还可以通过指针或成员运算符实现**隐式转换**：

```c++
pInv->hello(); // 假设 hello() 是 Investment 的成员函数
(*pInv).hello();
```

同样在自定义的资源管理类中，我们也可以实现管理对象到资源对象的显式和隐式转换：

```c++
// The C APIs is used to manage FontHandle resource
FontHandle getFont();
void releaseFont(FontHandle fh);

// Resource manager
class Font {
  private:
    FontHandle fh;
  public:
    explicit Font(FontHandle f): fh(f) { }
    ~Font() {
        releaseFont(fh);
    }
    // 通过 get() 实现显式转换
    FontHandle get() const { return fh; }
    // 实现隐式转换
    operator FontHandle() const { return fh; }
};

// The C APIs only accepts raw resource object FontHandle
void changeFontSize(FontHandle fh, int size);

Font f(getFont());
changeFontSize(f.get(), 20);  // 显式
changeFontSize(f, 20);  // 隐式
```

从上例可见隐式转换对客户更加友好。不过也潜藏着危险：

```c++
FontHandle fh;
{
	Font f(getFont());
	fh = f;  // 隐式转换，此时资源对象既可以被 f 也可以被 fh 访问
    
    // 离开区块时，资源对象被管理对象销毁
}

// 此时 fh 成为将引用无效资源对象
```

# Item 16: new 和 delete 要匹配

当创建对象时使用的是 `new` 则销毁对象时需要使用 `delete`，但如果创建对象时使用的是 `new name[]`，则销毁对象时应该使用 `delete [] name`。

错误示范：

```c++
std::string *ps1 = new std::string;
std::string *ps2 = new std::string[10];

delete [] ps1;  // error!
delete ps2;    // error!
```

在单个对象上调用 `delete [] name`，以及在数组对象上调用 `delete name` 都是未定义的行为。

先让我们来看 `new` 和 `delete` 到底干了什么？`new` 时，会分配内存并调用对象的构造函数，如果创建的是数组，会分配额外的内存用来记录数组的大小；`delete` 时，会调用对象的析构函数并释放内存。由于创建单个对象和数组的内存结构是不一样的，那 `delete` 是如何知道要处理的内存块是用来保存单个对象的，还是数组的呢？这就需要我们明确的通过 `delete [] name` 语法来告诉它。

一般来说，我们很容易保证这样去做，但如果遇到下面这种情形就会有点困惑了：

```c++
typedef std::string Address[4];

std::string *p = new Address;
delete p;  // error!
```

这里 `typedef` 会隐藏数据类型，最好不要对数组进行 `typedef`，取而代之可以用 `vector` ： `typedef std::vector<std::string> Address;`。

# Item 17: 以单独语句来添加资源对象

如果以复合语句添加资源对象，可能会导致资源泄漏，见下例：

```c++
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);

// 用复合语句添加资源对象
processWidget(std::shared_ptr<Widget>(new Widget), priority());
```

上面的复合语句有可能会导致资源泄漏，该语句可以拆分为三个原子操作：`new Widget `, `std::shared_ptr<Widget>` 和 `priority()`，由于 C++ 并不保证参数的评估顺序，因此可能会出现这样的执行顺序：`new Widget` -> `priority()` -> `std::shared_ptr<Widget>`，如果 `priority()` 抛出异常，那么之前创建的资源就会泄漏。

可以通过拆分语句来解决该问题：

```c++
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```





