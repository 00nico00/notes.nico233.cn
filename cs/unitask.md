## 1. 异步操作转化为 unitask

### 1.1 异步加载资源

```csharp
private async void OnClickLoadText()  
{  
    var loadOperation = Resources.LoadAsync<TextAsset>("test");  
    var text = await loadOperation;  
  
    TargetText.text = ((TextAsset) text).text; 
}
```

此处的 `loadOperation` 是一个 `ResourceRequest` ，而 `ResourceRequest` 其实就是一个异步操作 （ 继承自 `AsyncOperation`  ）。接着此处使用 `await` ，就会将其转换为 `Unitask` 的 `awaiter` 。代码如下：

```csharp
public static ResourceRequestAwaiter GetAwaiter(this ResourceRequest asyncOperation)  
{  
    Error.ThrowArgumentNullException(asyncOperation, nameof(asyncOperation));
    return new ResourceRequestAwaiter(asyncOperation);  
}
```

最终这个异步操作得到的就是 `awaiter` 里面返回的结果：

```csharp
public UnityEngine.Object GetResult()  
{  
    if (continuationAction != null)  
    {  
        asyncOperation.completed -= continuationAction;  
        continuationAction = null;  
        var result = asyncOperation.asset;  
        asyncOperation = null;  
        return result;  
    }  
    else  
    {  
        var result = asyncOperation.asset;  
        asyncOperation = null;  
        return result;  
    }  
}
```

最终返回的也就是我们这个  `ResourceRequestAwaiter` 里面的 `GetResult` ，也就是这个异步操作的 `asset`， 即 `asyncOperation.asset` 。

如果我们想将此处的功能从 `MonoBehaviour` 里面迁移到 `c#` 托管的类，此时可以这样：

```csharp
// 一个由 c# 托管的类
public class UniTaskAsyncSample_Base  
{  
    public async UniTask<Object> LoadAsync<T>(string path) where T: Object  
    {  
        var asyncOperation = Resources.LoadAsync<T>(path);  
        return (await asyncOperation);  
    }
}

// 修改后的使用代码
UniTaskAsyncSample_Base asyncUnitaskLoader = new UniTaskAsyncSample_Base();  
TargetText.text = ((TextAsset) (await asyncUnitaskLoader.LoadAsync<TextAsset>("test"))).text;
```

此处含有 `async` 修饰符，因此返回的必须是一个 `Task` 或者 `ValueTask` ，而 `UniTask` 本身就是一个 `ValueTask` 即**值任务**，

### 1.2 异步加载场景

