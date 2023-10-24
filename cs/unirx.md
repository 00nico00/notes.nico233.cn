## 1 OnNext & SubScribe

`OnNext` 和 `SubScribe` 都是在 `Subject` 中实现的方法，他们的行为分别如下：
+ `Subscribe` 在接收到消息时执行注册的函数
+ `OnNext` 将收到的消息传递给Subscribe注册的函数并执行

```c#
Subject<string> subject = new Subject<string>();  
  
subject.Subscribe(msg => Debug.Log("Subscribe1:" + msg + "\n"));  
subject.Subscribe(msg => Debug.Log("Subscribe2:" + msg + "\n"));  
subject.Subscribe(msg => Debug.Log("Subscribe3:" + msg + "\n"));  
  
subject.OnNext("hello 001");  
subject.OnNext("hello 002");  
subject.OnNext("hello 003");
```

放入 `start` 执行结果如下

```
Subscribe1:hello 001
Subscribe2:hello 001
Subscribe3:hello 001

Subscribe1:hello 002
Subscribe2:hello 002
Subscribe3:hello 002

Subscribe1:hello 003
Subscribe2:hello 003
Subscribe3:hello 003
```

![[Pasted image 20230930231317.png]]


## 2 一些示例

每一个的 `AddTo(this)` 只是为了使其生命周期和脚本一样，其实就是在运行的时候挂载了一个 `Observable Destory Trigger` 脚本
### 2.1 计时器

```c#
Observable.Timer(TimeSpan.FromSeconds(2f))  
    .Subscribe(_ => {  
        Debug.Log("2s");  
    })  
    .AddTo(this);
```

### 2.2 Update

```c#
Observable.EveryUpdate()  
    .Subscribe(_ => {  
        Debug.Log("update");  
    })  
    .AddTo(this);
```

### 2.3 Where

```c#
Observable.EveryUpdate()  
    .Where(_ => Input.GetMouseButtonDown(0))  
    .Subscribe(_ => {  
        Debug.Log("update");  
    })  
    .AddTo(this);
```

### 2.4 每秒输出一次

```c#
Observable.EveryUpdate()  
    .Sample(TimeSpan.FromSeconds(1f))  
    .Subscribe(_ => {  
        Debug.Log("update");  
    })  
    .AddTo(this);
```

### 2.5 实现 mvc架构

例如有一个 `int` 值，本来是希望在其改变时候输出一些日志

```c#
public class UniRxTest : MonoBehaviour  
{  
    private ReactiveProperty<int> index = new ReactiveProperty<int>(1);  
      
    private void Start() {  
        Func();  
  
        Observable.EveryUpdate()  
            .Where(_ => Input.GetMouseButtonDown(0))  
            .Subscribe(_ => {  
                index.Value++;  
            })  
            .AddTo(this);  
    }  
  
    private void Func() {  
        index
	        .Subscribe(_ => {  
                Debug.Log("count: " + index.Value);  
            })  
            .AddTo(this);  
    }  
}
```

这样每次点击**鼠标左键**，都会去执行 `index` 订阅的消息，此处 `ReactiveProperty` 在使用 `1` 初始化值的时候也会去触发订阅的方法。可以使用 `Skip` 跳过第一次初始化

```c#
index  
    .Skip(1)  
    .Subscribe(_ => {  
        Debug.Log("count: " + index.Value);  
    })  
    .AddTo(this);
```


## 3 生命周期增强版

比如要在物体隐藏的时候输出日志

```c#
gameObject.OnDestroyAsObservable()  
    .Subscribe(unit => {  
        Debug.Log("Disable");  
    })  
    .AddTo(this);
```

同理还有碰撞时，被鼠标点击时

## 4 AddTo

既然有事件的注册那么也会有取消注册

### 4.1 事件流主动取消注册

```c#
public class UniRxTest : MonoBehaviour  
{  
    public GameObject cube;  
  
    private void Start() {  
        Func();  
    }  
  
    private void Func() {  
        IDisposable disposable = cube.UpdateAsObservable()  
                                    .Subscribe(_ => {  
                                        Debug.Log("update");  
                                    });  
    }  
}
```

此时可以看到，返回的是一个 `IDisposable` 接口类型，也就是实现了 `Dispose` 这个方法，这个事件流现在就是可释放的，可以主动去释放这个事件。

### 4.2 AddTo

```c#
cube.UpdateAsObservable()  
    .Subscribe(_ => {  
        Debug.Log("update");  
    })  
    .AddTo(gameObject);
```

使用 `AddTo` 将这个事件流绑定到 `gameObject` 上面，当 `gameObject` 销毁时，也随之调用 `Dispose` 去释放绑定的事件

同时也可以使用 `CompositeDisposable` 去管理事件流的生命周期 
```c#
public class UniRxTest : MonoBehaviour  
{  
    public GameObject cube;  
    private CompositeDisposable _compositeDisposable;  
  
    private void Start() {  
        Func();  
    }  
  
    private void Func() {  
        _compositeDisposable = new CompositeDisposable();  
          
        cube.UpdateAsObservable()  
            .Subscribe(_ => {  
                Debug.Log("update");  
            })  
            .AddTo(_compositeDisposable);  
  
        // _compositeDisposable.AddTo(gameObject);  
    }  
  
    private void Update() {  
        if (Input.GetMouseButtonDown(0)) {  
            _compositeDisposable.Dispose();  
        }  
    }  
}
```
将事件流的生命周期交给 `CompositeDisposable` 管理后：
+ 可以直接调用 `Dispose` 去释放
+ 也可以再使用 `AddTo` 把生命周期绑定到其他物体或者脚本上面


## 5 Subject

类似于泛型委托 `Action` ，可以像这样注册事件
```c#
public class UniRxTest : MonoBehaviour  
{  
    private Subject<int> _subject = new Subject<int>();  
  
    private void Start() {  
        Func();  
    }  
  
    private void Func() {  
        _subject.Subscribe(_ => {  
                Debug.Log("log");  
            })  
            .AddTo(this);  
    }  
}
```

## 6 将事件转为 Observable

### 6.1 UnityEvent

```c#
public class UniRxTest : MonoBehaviour  
{  
    private UnityEvent _unityEvent;  
  
    private void Start() {  
        Func();  
    }  
  
    private void Func() {  
        _unityEvent.AsObservable()  
            .Subscribe(unit => {  
  
            }).AddTo(this);  
    }  
}
```

### 6.2 event Action

```c#
public class UniRxTest : MonoBehaviour  
{  
    private Action<int> _action;  
    private event Action _event;  
  
    private void Start() {  
        Func();  
    }  
  
    private void Func() {  
        Observable.FromEvent(action => _event += action, action => _event -= action)  
            .Subscribe(unit => {  
  
            }).AddTo(this);  
    }  
}
```

此处的 `_action` 就是 `Subscribe` 中注册的东西


## 7 MessageBroker

```c#
/// <summary>  
/// In-Memory PubSub filtered by Type.  
/// </summary>  
public class MessageBroker : IMessageBroker, IDisposable  
{  
    /// <summary>  
    /// MessageBroker in Global scope.    
    /// </summary>    
    public static readonly IMessageBroker Default = new MessageBroker();  
  
    bool isDisposed = false;  
    readonly Dictionary<Type, object> notifiers = new Dictionary<Type, object>();  
  
    public void Publish<T>(T message)  
    {  
        object notifier;  
        lock (notifiers)  
        {  
            if (isDisposed) return;  
  
            if (!notifiers.TryGetValue(typeof(T), out notifier))  
            {  
                return;  
            }  
        }  
        ((ISubject<T>)notifier).OnNext(message);  
    }  
  
    public IObservable<T> Receive<T>()  
    {  
        object notifier;  
        lock (notifiers)  
        {  
            if (isDisposed) throw new ObjectDisposedException("MessageBroker");  
  
            if (!notifiers.TryGetValue(typeof(T), out notifier))  
            {  
                ISubject<T> n = new Subject<T>().Synchronize();  
                notifier = n;  
                notifiers.Add(typeof(T), notifier);  
            }  
        }  
  
        return ((IObservable<T>)notifier).AsObservable();  
    }  
  
    public void Dispose()  
    {  
        lock (notifiers)  
        {  
            if (!isDisposed)  
            {  
                isDisposed = true;  
                notifiers.Clear();  
            }  
        }  
    }  
}
```

