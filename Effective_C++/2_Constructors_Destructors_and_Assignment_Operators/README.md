# Item 5: 编译器默认提供的成员函数

几乎每个类都有若干个构造函数、一个析构函数、以及一个 copy assignment 函数。如果自定义类没有定义这些函数，并且在代码中有使用到它们的场景的话，编译器会自动为自定义类生成它们。

例如我们定义了一个空类，并且创建、销毁以及赋值了它的对象，那么编译器会自动为该类创建一个 default 构造函数、一个 copy 构造函数、一个析构函数、一个 copy assignment 函数，并且它们的访问权限是 `public`。

```c++
// 定义空类
class Empty {};

// 编译器会为空类添加默认成员函数
class Empty {
  public:
    Empty();						// default constructor
    Empty(const Empty& e); 			// copy constructor
    ~Empty();						// destructor
    Empty& operator=(const Empty& rhs);	// copy assignment operator
    
};
```

其中生成的 default constructor 主要用于调用基类构造函数；生成的析构函数 destructor 主要用于调用基类的析构函数且 virtualness 跟基类保持一致；生成的 copy constructor 和 copy assignment 主要是将源对象的每一个 non-static 成员变量拷贝至目标对象。

再看一个例子：

```c++
template<typename T>
class A {
  private:
    std::string key;
    T value;
  public:
    A(const std::string& v, const T& v);
};
```

本例中由于已经存在构造函数，编译器不会再生成 default 构造函数，不过还是会生成 copy 构造函数和 copy assignment 函数：

```c++

```

编译器在以下几种情况下不会自动生成 copy assignment 函数：

- 成员变量是 const 或 reference
- 基类的 copy assignment 是 private 的

为什么成员变量是 const 或 reference，则编译器不会自动生成 copy assignment？

编译器自动生成的 copy assignment，其行为是拷贝源对象的 non-static 成员到目标对象，那么拷贝 reference？拷贝 const？是合法的吗？在 C++ 中，不允许让 reference 指向不同对象，也不允许改变常量的值。显然编译器如果生成 copy assignment 函数，其行为是不合法的，因此遇到此类情形，编译器就不会生成 copy assignment。

```c++
template<typename T>
class A {
  private:
    std::string& key;
    const T value;
  public:
    A(const std::string& k, const T& v);
};

A<int> a1("hello", 2);
A<int> a2("name", 3);
a1 = a2;		// 将会报错，没有 copy assignment 可用
```

为什么基类的 copy assignment 是 private 的，则编译器不会自动生成 copy assignment？

由于派生类无法调用基类的 copy assignment，因此需要自定义 copy assignment 来处理基类中 copy assignment 的逻辑。

# Item 6: 不想使用编译器提供的成员函数就该明确拒绝

假设某个类的每个实例对象都是独一无二的，因此我们不希望有途径可以创建对象的副本，并且希望在编译期间就可以指出试图创建副本的错误行为，如下：

```c++
class A { };

A a1();
A a2(a1);	// 希望编译期间报错
a1 = a2;	// 希望编译期间报错
```

也即我们希望该类不支持 copy 构造函数和 copy assignment 函数。不过即使我们不为此类定义它们，编译器也会自动生成 copy 构造函数和 copy assignment 函数，那要如何禁止这些创建副本的操作呢？

这里有两种方案：

- 将 copy 构造函数和 copy assignment 函数声明为 private 且不定义它们，则编译器不再自动生成它们，同时外部创建副本的操作会导致编译期错误。不过由于成员函数和友元函数中还是可以进行创建副本操作，而这类错误无法在编译期发现，只能在连接时由于 copy 构造函数和 copy assignment 函数未定义而报错。

  ```c++
  class A {
    private:
      A(const A& a);	// 仅声明，无定义
      A& operator=(const A& rhs);	// 仅声明，无定义
  };
  ```

- 创建一个基类，该基类中将 copy 构造函数和 copy assignment 函数声明为 private 且不定义它们，然后通过继承该基类来实现编译期间对创建副本的操作报错。

  ```c++
  class Uncopyable {
    protected:
      Uncopyable();
      ~Uncopyable();
    private:
      Uncopyable(const Uncopyable&);				// 声明为私有，且没有定义
      Uncopyable& operator=(const Uncopyable&);	// 声明为私有，且没有定义
  };
  
  // 由于基类已经声明了 copy constructor 和 copy assignment 函数，编译器不再自动生成它们
  class A: private Uncopyable { };
  ```

# Item 7: 多态基类声明虚析构函数

假设现在我们有多种计时工具，它们都继承自基类 `TimeKeeper`，同时我们还需提供一个工厂函数来为客户省去某个具体计时工具的创建过程，如下：

```c++
// 计时工具基类
class TimeKeeper {
  public:
    TimeKeeper();
    ~TimeKeeper();
};

// 各种计时工具
class AtomicClock: public TimeKeeper { };

class WaterClock: public TimeKeeper { };

class WristWatch: public TimeKeeper { };

// 工厂函数用于创建计时工具，假设它返回的计时器对象是动态内存分配的，因此需要客户在使用完后删除它
TimeKeeper* getTimeKeeper(std::string name);
```

客户使用时，通过工厂函数创建计时器对象，并用基类指针使用它，在用完后将其删除：

```c++
TimeKeeper* ptk = getTimeKeeper("atomic");
// ...
delete ptk;
```

这里我们要思考一个问题，用指向子类对象的基类指针可以成功删除该对象吗？由于指针是基类的，因此在试图进行删除该对象时，会调用当前指针类型（也即基类）的析构函数。而基类的析构函数仅仅负责释放它自己相关的成分，因此该子类对象中不属于基类的成分竟然就被漏掉了，这就造成了资源泄漏。

为了解决该问题，我们需要将基类析构函数设为 `virtual`，示例如下：

```c++
// 计时工具基类
class TimeKeeper {
  public:
    TimeKeeper();
    virtual ~TimeKeeper();
};
```

一般来说基类中，除了析构函数为 virtual 外，还有其它成员函数也是 virtual。或者反过来说，当类中有 virtual 成员函数时， 就应该为它指定一个 virtual  析构函数。还可以这样看，当基类支持多态时（即可以通过基类接口来处理派生类的对象），需要为基类定义 virtual 析构函数。

那当类中没有任何 virtual 成员函数时（不打算当作基类），是否也可以声明析构函数为 virtual？这样做有好处吗？

```c++
class Point {
  private:
    int x, y;
  public:
    Point();
    ~Point();
};

class VPoint {
  private:
    int x, y;
  public:
    Point();
    virtual ~Point();
};
```

上例中 `Point` 类占用两个 `int` 的空间，但 `VPoint` 需要更多的空间，因为 virtual 函数的实现，是通过维护一个虚表指针（virtual table pointer）来实现的，也是需要耗费空间的。可见，为没有 virtual 成员函数的类，指定virtual 析构函数，并没有任何好处。一般来说，只有当类中至少有一个 virtual 成员函数时，才会考虑将析构函数指定为 virtual。更加严格的讲，只有当基类具有多态性质时，才应该为它声明 virtual 析构函数（基类和多态两个性质缺一不可）。

除了自定义基类和派生类会遇到这个坑外，派生 STL 中带有 non-virtual 析构函数的类也会掉入这个坑，比如：自定义类继承自 `std::string`，然后该自定义类的对象使用 `std::string*` 类型的指针进行释放，同理会释放不完成，导致资源泄漏。STL 中带有 non-virtual 析构函数的类有：`string`, `vector`, `list`, `set` 等，不要试图去继承这些类。这是由于这些类的设计目的就不是作为基类而使用的，更不用说支持多态了。

如果想要将一个类声明为抽象类（abstract class），但又没有合适的成员函数该被声明为纯虚函数（pure virtual），此时可以将析构函数声明为纯虚函数，因为抽象类是不能实例化的基类，而基类应该有个虚析构函数：

```c++
class A {
  public:
    A();
    virtual ~A() = 0;
};

A::~A() { }  // 还需要提供空白定义
```

# Item 8: 析构函数不要抛出异常

C++ 虽然不禁止在析构函数中抛出异常，但一般我们不鼓励在析构函数中抛出异常。

假设析构函数中必须执行某个动作，而且该动作可能会抛出异常，该怎么办？例如，我们想要在析构函数中执行数据库连接的关闭：

```c++
class DBConnection {
  public:
    static DBConnection create();	// 打开数据库连接
    void close();					// 关闭数据库连接
};

class DBConnProxy {
  private:
    DBConnection db;
  public:
    ~DBConnProxy() {
        db.close();		// 析构函数中关闭连接
    }
};
```

客户可以将打开的数据库连接对象交给 `DBConnProxy` 对象，该对象析构时数据库连接会被自动关闭，这样客户省去了直接调用数据库连接对象 `close()` 成员的麻烦，也避免了遗忘关闭数据库：

```c++
DBConnProxy dbc(DBConnection::create());

// dbc 析构时，数据库连接对象自动关闭
```

如果析构函数中的 `db.close()` 抛出异常怎么办？为了避免析构函数抛出异常，可以采取以下两种方案：

- 析构函数中检测到异常时，强制结束程序

  ```c++
  DBConnProxy::~DBConnProxy() {
      try {
      	db.close();    
      } 
      catch (...) {
  		std::abort();
      }
  }
  ```

- 析构函数中检测到异常时，捕获并处理该异常

  ```c++
  DBConnProxy::~DBConnProxy() {
      try {
      	db.close();    
      } 
      catch (...) {
  		// 记录此异常
      }
  }
  ```

上面的两种方案都是在异常发生时，由开发者决定异常处理机制。有时将异常的处理权利移交给客户可能更合适：

```c++
class DBConnProxy {
  private:
    DBConnection db;
    bool closed;
  public:
    void close() {
        db.close();
        closed = true;
    }
    ~DBConnProxy() {
        if (!closed) {
            try {
                db.close();
            }
            catch (...) {
                // 处理异常
            }
        }
    }
};
```

此例中通过公开接口 `close()` 来给用户权利来处理关闭数据库连接期间引发的异常，如果客户放弃了该权利，析构函数仍然会关闭数据库连接并处理异常。

# Item 9: 绝不要在构造和析构函数中调用虚函数

股票交易有买进、卖出等，它们都有一个共同的交易基类，而所有的交易都应该被记录到审计系统中：

```c++
class Transaction {
  public:
    Transaction();
    ~Transaction();
    virtual void logTransaction() const = 0;	// pure virtual
};

Transaction::Transaction() {
    // construction
    
    // audit log, call a virtual function
    logTransaction();
}

class BuyTransaction: public Transaction {
  public:
    virtual void logTransaction() const;
};

class SellTransaction: public Transaction {
  public:
    virtual void logTransaction() const;
};
```

如果我们实例化 `BuyTransaction` 或 `SellTransaction` 将会发生什么？在实例化派生类时，基类成分会先被实例化，基类构造函数被调用，不过这里有个关键点：基类 `Transaction` 构造函数中调用了虚函数 `logTransaction()`，此虚函数是基类的版本而非派生类的版本。换句话说，基类构造期间虚函数绝不会下降到派生类。C++ 之所以这样规定，有以下几个理由：

- 基类构造时，派生类成员变量并没有初始化，如果虚函数是派生类的版本，那将会访问未初始化的派生类成员变量，显然是不合适的。
- 基类构造时，对象类型就是基类而非派生类，因此调用基类版本的虚函数也是合理的。对象在派生类构造函数调用之前，是不会成为一个派生类对象的。

上面的道理同样适用于析构函数，绝对不要在析构函数中调用虚函数。不仅仅是直接调用虚函数不行，间接调用也是不合理的。而且这类错误很难被发现：如果基类虚函数没有定义的话，连接器可能会报错；如果基类虚函数有定义的话，系统正常被编译连接和运行。

那么如何保证每个交易对象被创建时，就会记录到审计系统中呢？不要依赖虚函数的多态特性让不同交易对象实现各自的审计逻辑，而应该让派生类将审计信息打包后传递给基类构造函数，如下：

```c++
class Transaction {
  public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const;	// non-virtual
};

Transaction::Transaction(const std::string& logInfo) {
    
    // audit log
    logTransaction(logInfo);
}

class BuyTransaction: public Transaction {
  public:
    BuyTransaction(params): Transaction(createLogString(params)) { }
  private:
    // 创建该派生类的审计日志信息
    static std::string createLogString(params);  // static 确保不会访问 non-static 成员变量（它们还未初始化）
};
```

# Item 10: 赋值运算函数返回对自身的引用

要实现链式赋值，如：`a=b=c=1`，自定义类的赋值运算函数需要返回运算符左侧操作数的引用（即对象自身的引用），示例：

```c++
class A {
  public:
    A& operator=(const A& rhs) {  	// 返回类型是一个引用
        // ...
        return (*this);				// 返回当前对象，即左操作数
    }
};
```

此外其它赋值运算也遵从该准则，如：`+=`， `-=`， `*=` 等。该准则只是一个约定而已，即使不遵循它，也一样可以通过编译。

# Item 11: 赋值运算函数中处理自我赋值问题

什么是自我赋值？

```c++
class A;

A a1;
a1 = a1;

A l[10];
int i=3, j=3;
l[i] = l[j];

A* pa = &a1;
A* pb = &a1;
*pa = *pb;
```

从上例可以看出，由于别名的存在自我赋值以各种方式存在，是不可避免的。

那么当某个对象赋值给它自己时，会发生什么，或者说我们在处理赋值时，应该要为自我赋值注意些什么？

```c++
class Bitmap {};
class Widget {
  public:
    Widget& operator=(const Widget& rhs);
  private:
    Bitmap* bp;
};

Widget& Widget::operator=(const Widget& rhs) {
    delete bp;
    bp = new Bitmap(*(rhs.bp));
    return *this;
}
```

如果上例中的 `rhs` 就是当前对象呢？先把当期对象的 `bp` 删除，再拷贝它，最后返回的指针指向了已经被删除的对象，显然逻辑是错误的。我们可以通过增加自我测试来改进它：

```c++
Widget& Widget::operator=(const Widget& rhs) {
    if (this == &rhs) return *this;
    delete bp;					// 删除当前对象的 bp
    bp = new Bitmap(*(rhs.bp)); // 拷贝 rhs 对象的 bp
    return *this;
}
```

虽然现在自我赋值是安全了，但还有个问题没解决：如果创建新 `Bitmap` 对象抛出异常失败呢？由于在创建新对象之前，先删除了原来的 `Bitmap` 对象，因此新对象创建失败将导致 `bp` 指向一个已经被删除的对象。这个问题叫做异常安全问题。要解决该问题需要这样：

```c++
Widget& Widget::operator=(const Widget& rhs) {
	Bitmap *tmp = bp;
    bp = new Bitmap(*(rhs.bp));
    delete tmp;
    return *this;
}
```

最后的解决方案同时具备自我赋值安全性和异常安全性。

# Item 12: 复制对象时要完整

> In well-designed object-oriented systems that encapsulate the internal
> parts of objects, only two functions copy objects: the aptly named
> copy constructor and copy assignment operator. We’ll call these the
> copying functions.

## 复制所有成员变量

这里有一个顾客类：

```c++
void logCall(const std::string& funcName);

class Customer {
  public:
    Customer(const Customer& rhs);
    Customer& operator=(const Customer& rhs);
  private:
    std::string name;
    long timestamp;
};

Customer::Customer(const Customer& rhs): name(rhs.name) {
    logCall("copy constructor");
}

Customer& Customer::operator=(const Customer& rhs) {
    name = rhs.name;
    logCall("copy assignment operator");
    return *this;
}
```

上面的问题在哪里？copy 构造函数和 copy 赋值函数都没有复制成员 `timestamp`，但这并不是语法错误，编译器不会为此报错。改进如下：

```c++
Customer::Customer(const Customer& rhs): name(rhs.name), timestamp(rhs.timestamp) {
    logCall("copy constructor");
}

Customer& Customer::operator=(const Customer& rhs) {
    name = rhs.name;
    timestamp = rhs.timestamp;
    logCall("copy assignment operator");
    return *this;
}
```

每次为类增加一个新的成员变量时，也要同时修改 copying 函数使得它们也会复制新成员。

## 复制基类成员变量

再来看一个例子：

```c++
class PriorityCustomer: public Customer {
  public:
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer& operator=(const PriorityCustomer& rhs);
  private:
    int priority;
};

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs): priority(rhs.priority) {
	    
}

PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs) {
    priority = rhs.priority;
    return *this;
}
```

这个例子中的问题在于，派生类的复制函数仅仅处理了自己声明的成分，而没有处理继承自基类的成分，改正如下：

```c++
// 成员初始化列表中调用基类 copy 构造函数
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs): Customer(rhs), priority(rhs.priority) {
	    
}

PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs) {
    // 调用基类的 copy 赋值函数
    Customer::operator=(rhs);
    priority = rhs.priority;
    return *this;
}
```

## 抽取 copy  构造和 copy 复制函数的共同逻辑

copy  构造和 copy 复制函数有许多逻辑是共用的，不过不推荐用其中一个调用另一个来避免重复，因为这样做是不合理的。一般我们会将共用的逻辑放在一个 `init` 的 private 函数中，然后分别在 copy  构造和 copy 复制函数中调用它。















