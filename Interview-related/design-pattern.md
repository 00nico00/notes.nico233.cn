## 1.设计模式的7大设计原则有哪些，讲一下依赖反转原则
1. 单一职责原则：在类的级别上，一个类只负责一项职责；在方法的级别上，一个方法只做一件事。
2. 开放-关闭原则：对扩展开放，对修改关闭，表示软件实体 (类、模块、函数等等) 应该是可以被扩展的，但是不可被修改。例如有一个 `ComputerFactory` 方法，其函数签名为  `Computer ComputerFactory(ComputerType type)` ，而其中是使用判断类型来返回的，那么此就违反了开闭原则，正确做法是提供一个 `IComputerFactory` 接口，让不同的类型厂家去实现对应方法。
3. 里氏替换原则：子类可以扩展父类的功能，但不能改变父类原有的功能。在子类中不要重写父类的方法（重写抽象类的抽象方法除外）。如果 `B` 想要继承 `A` 类的方法并且重写，那就再定义一个更高层的 `Base` 基类，再使得 `A` 和 `B` 去继承 `Base` ，此时 `B` 想使用 `A` 的东西可以任意**聚合**，**组合**，**依赖**了。
4. 依赖反转原则：高层模块不应该依赖低层模块，抽象不应当依赖于细节；细节应当依赖于抽象。通俗来讲就是“面向接口编程”。
5. 接口隔离原则：客户端不应该依赖它不需要的接口；**一个类对另一个类的依赖应该建立在最小的接口上。**
6. 迪米特法则/最少知道原则：每个类 `A` 都避免不了与其他类 `B` 产生关系，但我们让这种关系越小越好。即类 `A` 对类 `B` 的内部了解的越少越好，如果类 `A` 要用到类 `B` ，最好只需要调用类 `B` 提供的 `public` 方法即可。一般做法为：只允许在一个类的**成员变量（组合）、方法参数（依赖）、方法返回值（依赖）** 处使用另一个类的对象。 **不允许把另一个类的对象当做自己的局部变量**。
7. 合成复用原则：能使用组合、依赖就不使用继承

### 1.1 依赖反转补充
给出如下一个场景：
```csharp
public class Book {
    public string GetContent() {
        return "书的内容";
    }
}

public class Mother {
    public void say(Book book) {
        Debug.log(book.GetContent());
    }
}
```
妈妈给孩子讲故事，拿到一本书就可以照着书读了，但是突然有一天给了她报纸
```csharp
public class Newspaper {
    public string GetContent() {
        return "报纸的内容";
    }
}
```
虽然此时，报纸和书本都有 `GetContent` 也就是妈妈都能知道这个文本载体的文本是什么。但是此处，拿到报纸并不能读出来，这是很难理解的事情。此时 `Mother` 就是高层模块，可以读的文本就是底层模块，此处 `Mother` 直接依赖于 `Book` ，就违反了依赖反转原则。因此我们就需要定义一个 `IReader` 接口，来防止这种高层模块和低层模块的强依赖关系。
```csharp
public interface IReader {
    void GetContent();
}

public class Book : IReader {
    public string GetContent() {
        return "书的内容";
    }
}

public class Newspaper : IReader {
    public string GetContent() {
        return "报纸的内容";
    }
}

public class Mother {
    public void Say(IReader reader) {
        Debug.log(book.GetContent());
    }
}
```
### 1.2 接口隔离补充
有如下场景：
```csharp
public interface Interface1 {
    void Operator1();
    void Operator2();
    void Operator3();
    void Operator4();
    void Operator5();
}

// 存在 A, B 两类实现了 Interface1 的所有接口
// 定义省略

// 此处存在 class C, D 通过 Interface1 分别依赖 A, B
public class C {
    public void DependA1(Interface1 i) {
        i.Operator1();
    }
    public void DependA2(Interface1 i) {
        i.Operator2();
    }
    public void DependA3(Interface1 i) {
        i.operator3();
    }
}

public class D {
    public void DependB1(Interface1 i) {
        i.Operator1();
    }
    public void DependB4(Interface1 i) {
        i.Operator4();
    }
    public void DependB5(Interface1 i) {
        i.operator5();
    }
}
```
很明显，以上代码违反了接口隔离原则。类 `C` 只需要 `A` 的 1/2/3 个实现，但是 `A` 却实现了五个接口，同理 `D` 对 `B` 也是一样的。因此此处需要拆分出**最小接口**
```csharp
public interface Interface1 {
    void operator1();
}

public interface Interface2 {
    void operator2();
    void operator3();
}

public interface Interface3 {
    void operator4();
    void operator5();
}

// 此时再使用 A, B 去实现相应需要的接口
public class A : Interface1, Interface2 {
    // ...
}

public class B : Interface1, Interface3 {
    // ...
}
```

## 2.unity不同脚本去实现单例模式，那么这个同步会被破坏的情况怎么解决?

使用 private 构造函数，防止外部脚本直接实例化
