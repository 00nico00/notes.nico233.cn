## 1.谈一下 c++ 的多态

`c++` 的多态分为静态多态和动态多态：
+ 静态多态：是编译期多态，通过函数重载或者函数模板实现
+ 动态多态：是运行期多态，主要指虚函数调用，通过虚函数表实现

## 2.虚函数表存储位置，函数指针如何访问虚函数表

虚函数表存储在 `.rodata` ，即**只读数据段**，虚函数存储在 `.text` ，即**代码段**，虚函数指针存放位置和对应对象存储位置一样
`c++` 标准并没有规定虚函数指针存放的位置，通常编译器实现是存放在对象的开头，因此，我们可以将对象强转为一个 `unsigned long long pointer` 此处叫 `p` ，接着再使用一个 `unsigned long long pointer` 此处叫 `arr` 去接 `p` 指针解引用的结果（需要强转），此时再将 `arr` 使用下标访问，将对应的虚函数转成相同签名的函数指针即可访问
```cpp
struct A {
	virtual void a() { std::cout << "A a()" << std::endl; }
	virtual void b() { std::cout << "A b()" << std::endl; }
	virtual void c() { std::cout << "A c()" << std::endl; }
	int x, y;
};

using u64 = unsigned long long;
using func = void(*)();

A a{};
u64* p = (u64*)&a;
u64* arr = (u64*)*p;

func fa = (func)arr[0];
func fb = (func)arr[1];
func fc = (func)arr[2];

fa(); // A a()
fb(); // A b()
fc(); // A c()
```

## 3. c++11 之后用哪三种智能指针，分别有什么应用场景，解决了什么问题

`c++11` 之后增添了三种智能指针：
+ `std::unique_ptr` ：适用于独占所有权的情况，即一个对象只能由一个指针拥有。解决问题：提供了独占的所有权管理，并且使用 `RAII` （资源分配即初始化）管理动态分配的资源，减少了内存泄漏的风险。
+ `std::shared_ptr` ：适用于多个指针共享同一个对象所有权的情况。解决问题：其使用引用计数来允许多个指针共享对同一对象的所有权。解决了传统指针的悬挂引用和内存泄漏的问题。
+ `std::weak_ptr` ：适用于解决 `std::shared_ptr` 可能导致的循环引用问题。当 `std::shared_ptr` 两个对象相互引用时，就会导致循环引用内存无法释放。 `std::weak_ptr` 是一种弱引用指针，其允许观察 `std::shared_ptr` 但是又不会增加计数

## 4. 构造函数、析构函数、static成员函数能否被设置为虚函数

构造函数不能为虚函数，析构函数可以为虚函数且实现多态时应将其设置为虚函数，static 成员函数不能为虚函数。

## 5. 构造函数为什么不能为虚函数

+ 从存储角度看：是虚函数的调用需要。在创建对象的时候，首先是为其分配内存，如何才调用构造对象在此内存上构造出对象。而这个流程中也包括初始化 `vptr` ，首先此指针也是被分配了内存，被存储在对象的内存空间当中，接着在构造函数当中才进行初始化工作，让其指向正确的虚函数表。如果构造函数为虚函数的话，那么 `vptr` 也没有初始化，仍为随机值，也没有指向正确的虚函数表来调用为虚函数的构造函数了。

