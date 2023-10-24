## 1 委托

委托就是一个密封类，其对象维护着一个可以引用一个或多个方法的字段，有点类似于 `c/c++` 中的函数指针，为了使委托去完成某种动作，必须满足4个条件
+ 声明委托类型
+ 必须有一个方法包含了要执行的代码
+ 必须创建一个委托实例
+ 必须调用（invoke）委托实例

声明一个委托类型
```csharp
delegate void StringProcessor(string input);
```
其实际上是从 `System.MulticastDelegate` 派生的类型，后者又是从 `System.Delegate` 派生

实例化委托
```csharp
delegate void StringProcessor(string input);

// 1
StringProcessor stringProcessor = new StringProcessor(p1.Say);

// 2
StringProcessor stringProcessor = p1.Say;

// 3 匿名委托
StringProcessor stringProcessor = delegate (string input) {
};

// 4 lambda
StringProcessor stringProcessor = (string input) => {
};
```

调用委托
```csharp
delegate void StringProcessor(string input);

// 1
StringProcessor("input_string");

// 2
StringProcessor.Invoke("input_string");
```

委托实例实际有一个操作列表与之关联。这称为委托实例的**调用列表**（ `invocation list` ）
`System.Delegate` 类型有两个静态方法负责此操作：
+ `Combine` : 负责将两个委托实例的调用列表连接到一起
+ `Remove` : 负责从一个委托实例中删除另一个实例的调用列表
其实他们都是在**创建新的委托示例**，委托的实例是**不易变的**，同 `string` 的实例一样，注意，如果试图将 `null` 和委托实例合并到一起，`null` 将被视为带有空调用列表的一个委托

### 1.1 委托的本质

委托本质实际上是一个密封类，继承自 `MulticastDelegate` ，它内部维护了一个字段，该字段指向一个或多个方法。这个字段是一个代表方法的引用
+ `MulticastDelegate` ：即多播委托，一个委托可以装载多个相同签名的方法，委托被调用时，方法依次执行
在 `System.Delegate` 当中有这四个成员变量

```csharp
internal object? _target; 

internal object? _methodBase; 

internal IntPtr _methodPtr;

internal IntPtr _methodPtrAux;
```

其中，主要关注 `_target` 和 `_methodPtr` 两个数据成员
+ `_target` ：表示要调用的对象，如果委托表示**实例方法**，则为当前委托对其调用实例方法的对象；如果委托表示**静态方法**，则为 `null`
+ `_methodPtr` ：此变量的类型为 `IntPtr` ，在 `c#` 中是对指针的封装，一般来表示指针或者句柄，因此 `_methodPtr` 是一个指向将要调用的方法的指针

## 2 事件

事件就是一种特殊的委托，在委托实例化的时候加上 `event` 关键字即可声明一个事件
```csharp
public event EventHandler OnTrigger;

// 此处的 EventHandler 就是个系统提供的委托类型
public delegate void EventHandler(object? sender, EventArgs e);
```

事件通常要和类结合起来使用，因为其**只能在定义事件的类中被调用**。

### 2.1 EventHandler & 订阅模式

```csharp
class Foo {
	public event EventHandler<OnTriggerArgs> OnTrigger;

	public class OnTriggerArgs : EventArgs {

	}

	public void Trigger() {
		OnTrigger?.Invoke(this, new OnTriggerArgs());
	}
}

class Bar {
	private Foo foo = new Foo();

	public Bar() {
		foo.OnTrigger += Foo_OnTrigger;
	}

	private void Foo_OnTrigger(object? sender, Foo.OnTriggerArgs e) {
		
	}
}
```

以上就是一个 `EventHandler` 类型事件的创建与调用：
+ `EventHandler` 的泛型类型参数就是需要传递的参数类型，必须继承自 `EventArgs`
+ 在调用其的时候，需要传入两个参数：`sender` 就是事件的发送者，`e` 就是传递进去的参数
+ 包含事件的类用于发布事件，称为发布器类，其他接受该事件的类称为订阅器类，事件使用发布-订阅模型

## 3 值类型和引用类型

通常，`class` 声明的是引用类型，`struct` 声明的是值类型，有以下几种特殊情况：
+ **数组**是引用类型，即使元素是值类型
+ `enum` 是值类型
+ `delegate` 是引用类型
+ `interface` 是引用类型，但是可由值类型实现

内存分布：
+ 值类型的值不一定存储在 `stack` 上，当一个值类型被定义为引用类型的字段时，该字段的值将与对象的其他数据一起存储在堆上。
+ 引用类型实例（对象）存储在 `heap` 上

两者的特性：
+ 值类型不可以派生出其他类型，因此值类型不需要额外的信息来描述**实际**是什么类型
+ 引用类型每个对象的开头都包含一个数据块，用于标明对象的实际类型。引用本身并不知道对象的类型

## 4 装箱和拆箱

```csharp
int i = 5;
object o = i;   // 装箱
int j = (int)o; // 拆箱
```

在**装箱**的时候，运行时将在堆上创建一个包含值（5）的对象（它是一个普通对象）。o的值是对该新对象的一个引用。**该对象的值是原始值的一个副本，改变i的值不会改变箱内的值。**

**拆箱**时必须告诉编译器将 `object` 拆箱成什么类型。如果使用了错误的类型（比如 `o` 原先被装箱成 `unit` 或者 `long` ，或者根本就不是一个已装箱的值），就会抛出一个 `InvalidCastException` 异常。同样，拆箱也会复制箱内的值，在赋值之后，`j` 和该对象之间不再有任何关系。

## 5 lambda

### 5.1 泛型委托类型

首先的了解一下泛型 `Func` 委托类型，其提供了一些提供了一些好用的预定义泛型类型

```csharp
TResult Func<TResult>()
TResult Func<T, TResult>(T arg)
TResult Func<T1, T2, TResult>(T1 arg1, T2 arg2)
TResult Func<T1, T2, T3, TResult>(T1 arg1, T2 arg2, T3 arg3)
TResult Func<T1, T2, T3, T4, TResult>(T1 arg1, T2 arg2, T3 arg3, T4, arg4)

// 其实 Func<int, double> 就等价于以下的委托类型
public delegate double SomeDelegate(int arg)
```

如果想返回 `void` ，则可以使用 `Action<...>` 系列的委托

### 5.2 lambda语法

```csharp
// 1
( /* 显示类型的参数列表 */ ) => { /* 语句 */ }

// 2
( /* 显示类型的参数列表 */ ) => /* 表达式 */

// 3 此处为隐式推断类型，一般需使用泛型委托指定类型
( /* 隐式类型的参数列表 */  ) => /* 表达式 */
// eg:
Func<string, int> func = (text) => text.Length;

// 4
/* 参数名 */ => /* 表达式 */
// eg:
Func<string, int> func = text => text.Length;
```

## 6 匿名方法中的捕获变量

```csharp
string str = "111";

var x = delegate () {
	Console.WriteLine(str);
	str = "222";
};

x();
Console.WriteLine(str);
str = "444";
x();

// output:
// 111
// 222
// 444
```

此例子解释了，匿名方法可以捕获外部变量
+ `outer variable` ：即外部变量，是指在**作用域内**包括匿名方法的**局部变量和参数**（不包括 `ref` 和 `out` 参数）。在类的实例成员的匿名方法中，`this` 引用也被认为是一个外部变量
**在匿名方法外部对变量的更改在匿名方法内部是可见的**，反之也成立

### 6.1 捕获外部变量的用处

捕获变量能简化避免专门创建一些类来存储一个委托需要处理的信息（除了作为参数传递的信息之外）。
如以下例子：**假定你有一个人物列表，并希望写一个方法来返回包含低于特定年龄的所有人的另一个列表。**
```csharp
List<Person> FindAllPersonYoungerThan(List<Person> people, int limit) {
	return people.FindAll(delegate(Person person) {
		person.Age < limit;
	});
}
```

### 6.2 延长变量的生存期

规则：对于一个捕获变量，只要还有任何委托实例在引用它，它就会一直存在。
```csharp
static Action CreateDelegateInstance() {
	int counter = 5;

	Action ret = delegate {
		Console.WriteLine(counter);
		counter++;
	};
	ret(); // output 5

	return ret;
}
...
var x = CreateDelegateInstance();
x(); // output 6
x(); // output 7
```
按照我们常规的思维，`counter` 是在栈上的，方法返回后，`CreateDelegateInstance()` 对应的栈帧被清除了，`counter` 也就随之消失了，可此处的 `counter` 还是之前的那个
因此，编译器创建了一个额外的类来容纳变量，`CreateDelegateInstance()` 方法拥有对该类一个实例的引用，这个实例和其他实例一样在堆上。如果只捕获了 `this` ，就不需要额外的类型了。

#### 6.2.1 使用多个委托来捕捉多个变量实例

```csharp
List<Action> list = new List<Action>();

for (int index = 0; index < 5; index++) {
	int counter = index * 10;
	list.Add(delegate {
		Console.WriteLine(counter);
		counter++;
	});
}
foreach (Action action in list) {
	action(); // output 0 10 20 30 40
}

list[0](); // 1
list[0](); // 2
list[0](); // 3
list[0](); // 4

list[1](); // 11
```


## 7 迭代器

### 7.1 手写迭代器

首先给出 `IEnumerable` 和 `IEnumerator` 的定义
```csharp
// IEnumerable
public interface IEnumerable {
	IEnumerator GetEnumerator();
}

// IEnumerator
public interface IEnumerator {
	object Current { get; }

	bool MoveNext();

	void Reset();
}
```

此处我们以嵌套类的形式实现这些接口，即可实现最简单的手写迭代器
```csharp
class IterationSample : IEnumerable {
	object[] values;
	int startingPoint;

	public IterationSample(object[] values, int startingPoint) {
		this.values = values;
		this.startingPoint = startingPoint;
	}

	public IEnumerator GetEnumerator() {
		return new IterationSampleIterator(this);
	}

	class IterationSampleIterator : IEnumerator {
		IterationSample parent;
		int position;

		internal IterationSampleIterator(IterationSample parent) {
			this.parent = parent;
			position = -1;
		}

		public bool MoveNext() {
			if (position != parent.values.Length) {
				position++;
			}
			return position < parent.values.Length;
		}

		public object Current {
			get {
				if (position == -1 || position == parent.values.Length) {
					throw new InvalidOperationException();
				}
				int index = position + parent.startingPoint;
				index %= parent.values.Length;
				return parent.values[index];
			}
		}

		public void Reset() {
			position = -1;
		}
	}
}

string[] values = { "1", "2", "3", "4", "5" };
IterationSample collection = new IterationSample(values, 2);
foreach (var val in collection) {
	Console.WriteLine(val);
}
// ouput 3 4 5 1 2
```

### 7.2 使用 yield 简化迭代器

我们可以使用如下代码替换上面的 `GetEnumerator()` 及 `IterationSampleIterator`
```csharp
public IEnumerator GetEnumerator() {
	for (int index = 0; index < values.Length; index++) {
		yield return values[(index + startingPoint) % values.Length];
	}
}
```

`yield return` 也有一些限制：
+ 如果存在任何 `catch` 代码块，则不能在 `try` 代码块中使用 `yield return` 
+ 不能在 `finally` 代码块中使用 `yield return` 或者 `yield break` 

此处看似是顺序执行，实际上是编译器自动生成创建了一个**状态机**，`yield return` 这句话告诉编译器，此处是实现一个**迭代器块**的方法。如果方法声明的返回类型是非泛型接口，那么迭代器块的生成类型（ `yield type` ）是 `object` ，否则就是泛型接口的类型参数。

当编译器看到迭代器块的时候，会为状态机创建一个嵌套类型。所创建的类类似于我们之前用普通方法实现的类。其含有：
+ 初始状态
+ `MoveNext` ，并且执行到 `MoveNext` 之前（即执行到 `yield return` 之前），它需要执行`GetEnumerator` 方法中的代码
+ `Current` ，并且在使用时，必须返回上一个生成的值
+ 其必须知道**何时完成**生成值的操作，即 `MoveNext` 返回 `false`

只要使用了 `foreach` ，迭代器块中的 `finally` 代码就会照常进行，因为调用其对象的 `Dispose` 方法，会触发迭代器块中的 `finally` 代码。而 `foreach` 则会在循环结束时调用对象的 `Dispose` 方法


## 8 反射

反射可以通过类名的字符串来创建类，可以通过函数名的字符串和属性名的字符串，来调用类下的函数和属性。

### 8.1 访问或修改类型的实例、静态字段

```csharp
namespace csharp_test {
	public class MyClass {
		public int myField;
		public static int myStaticField;
	}
}

// 访问或修改类型的实例字段myField
MyClass myObj = new MyClass() { myField = 1 };
Type myType = typeof(MyClass);                    // 获取类型
Console.WriteLine(myType);                        // csharp_test.MyClass
FieldInfo fieldInfo = myType.GetField("myField"); // 获取类型中指定的字段信息
Console.WriteLine(fieldInfo);                     // Int32 myField
int value = (int)fieldInfo.GetValue(myObj);
Console.WriteLine(value);                         // 1
fieldInfo.SetValue(myObj, 2);                     // 给实例字段赋值

// 访问或修改类型的静态字段 myStaticField
Type myType = typeof(MyClass);
FieldInfo staticFieldInfo = myType.GetField("myStaticField");
Console.WriteLine(staticFieldInfo.GetValue(null)); // 0
staticFieldInfo.SetValue(null, 2);
Console.WriteLine(MyClass.myStaticField);          // 2
```

### 8.2 访问或修改类型的实例、静态属性

```csharp
namespace csharp_test {
	public class MyClass {
		public int MyProperty { get; set; }
		public static int MyStaticProperty { get; set; }
	}
}

// 访问或修改类型的实例属性 MyProperty
MyClass myObj = new MyClass() { MyProperty = 1 };
Type myType = typeof(MyClass);
PropertyInfo propertyInfo = myType.GetProperty("MyProperty");
Console.WriteLine(propertyInfo);                      // Int32 MyProperty
Console.WriteLine((int)propertyInfo.GetValue(myObj)); // 1
propertyInfo.SetValue(myObj, 2);

// 访问或修改类型的静态属性 MyStaticProperty
Type myType = typeof(MyClass);
PropertyInfo staticPropertyInfo = myType.GetProperty("MyStaticProperty");
Console.WriteLine(staticPropertyInfo); // Int32 MyStaticProperty
Console.WriteLine(staticPropertyInfo.GetValue(null)); // 2
staticPropertyInfo.SetValue(null, 2);
```

**注意**：在使用反射给属性赋值的时候，如果该属性没有 `get` 访问器，则会抛出异常 `ArgumentException`

### 8.3 调用类型的方法

```csharp
namespace csharp_test {
	public class MyClass {
		public void MyFunc(int num) {
			Console.WriteLine($"call MyFunc, parameter: {num}"); ;
		}

		public static void MyStaticFunc(int num) {
			Console.WriteLine($"call MyStaticFunc, parameter: {num}");
		}
	}
}

// 调用类型的实例方法 MyFunc
MyClass myObj = new MyClass();
Type myType = typeof(MyClass);
MethodInfo methodInfo = myType.GetMethod("MyFunc");
methodInfo.Invoke(myObj, new object[] { 10 });
// call MyFunc, parameter: 10

// 调用类型的实例方法 MyStaticFunc
Type myType = typeof(MyClass);
MethodInfo staticMethodInfo = myType.GetMethod("MyStaticFunc");
staticMethodInfo.Invoke(null, new object[] { 20 });
// call MyStaticFunc, parameter: 20
```

### 8.4 调用类型的构造函数同时创建实例

```csharp
namespace csharp_test {
	public class MyClass {
		public MyClass() {
            Console.WriteLine("constructor MyClass()");
        }

		public MyClass(int num) {
            Console.WriteLine($"construct MyClass(int), num: {num}");
        }
	}
}

// 调用无参的构造函数
Type myType = typeof(MyClass);
// 获取类型中指定的构造函数信息，传入该构造函数的参数列表的类型数组，无参传空数组
ConstructorInfo constructorInfo = myType.GetConstructor(new Type[] { });
var myObj = constructorInfo.Invoke(null) as MyClass; // 无参传入 null
// constructor MyClass()

// 调用有参的构造函数
Type myType = typeof(MyClass);
ConstructorInfo constructorInfo = myType.GetConstructor(new Type[] { typeof(int) });
var myObj = constructorInfo.Invoke(new object[] { 20 }) as MyClass;
// construct MyClass(int), num: 20
```

### 8.5 类型信息

类型信息（ `Type Information` ）用来表示类型声明的信息，通过抽象基类 `System.Type` 的实例化对象存储这些信息。当使用反射的时候，`CLR` 获取指定类型的 `Type` 对象，通过这个对象就能访问该类型的任何信息
以下是几种获取指定类型 `Type` 对象的方法：
```csharp
MyType myobj = new Mytype();

// 1
Type myType = typeof(myobj);

// 2
Type myType = myobj.GetType();

// 3
Type myType = Type.GetType("MyType"); // 如果有命名空间，需要在类名前加上

// 4
// 其中 assembly 是当前程序集实例
Assembly assembly = Assembly.GetExecutingAssembly();
Type t = assembly.GetType("csharp_test.MyClass");
```

有以下几点注意：
+ 