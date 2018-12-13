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



