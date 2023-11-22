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

```csharp
private async void OnClickLoadScene()  
{  
    await SceneManager.LoadSceneAsync("UniTaskTutorial/BaseUsing/Scenes/TargetLoadScene").ToUniTask(  
        (Progress.Create<float>(  
            (p) =>  
            {  
                LoadSceneSlider.value = p;  
                if (ProgressText != null)  
                {  
                    ProgressText.text = $"读取进度{p * 100:F2}%";  
                }  
            })));  
}
```

此处的代码示例目的是异步加载一个场景，并且获取其加载进度反应到进度条上面。

此处将这个异步操作转换为 `UniTask` 之后，在其中添加一个回调 `Progress.Create<float>` ，即可拿到这个场景加载的进度。

### 1.3 网络请求的异步操作

```csharp
private async void OnClickWebRequest()  
{  
    var webRequest =  
        UnityWebRequestTexture.GetTexture(  
            "https://s1.hdslb.com/bfs/static/jinkela/video/asserts/33-coin-ani.png");  
    var result = (await webRequest.SendWebRequest());  
    var texture = ((DownloadHandlerTexture) result.downloadHandler).texture;  
    int totalSpriteCount = 24;  
    int perSpriteWidth = texture.width / totalSpriteCount;  
    Sprite[] sprites = new Sprite[totalSpriteCount];  
    for (int i = 0; i < totalSpriteCount; i++)  
    {  
        sprites[i] = Sprite.Create(texture,  
            new Rect(new Vector2(perSpriteWidth * i, 0), new Vector2(perSpriteWidth, texture.height)),  
            new Vector2(0.5f, 0.5f));  
    }  
    float perFrameTime = 0.1f;  
    while (true)  
    {  
        for (int i = 0; i < totalSpriteCount; i++)  
        {  
            await Cysharp.Threading.Tasks.UniTask.Delay(TimeSpan.FromSeconds(perFrameTime));  
            var sprite = sprites[i];  
            DownloadImage.sprite = sprite;  
        }  
    }  
}
```
此处的 `await` 其实也是在获得一个 `UnityWebRequestAsyncOperationAwaiter` ，和第一个加载资源的操作也是有异曲同工之妙了。

代码中间部分就是一个很简单的分割图片的算法。

代码最后的循环是将每帧 `sprite` 依次赋值，构成类似于动画的样子。此处虽然是死循环，但是有 `await` 因此是非阻塞的完全没问题。

## 2.Delay & Wait

