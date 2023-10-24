## 一个基本的实现初始化和重置功能的对象池

```csharp
public sealed class ObjectPool<T>  
    where T : class, new()  
{  
    private Queue<T> m_objectQueue;  
  
    private Action<T> m_resetAction;  
    private Action<T> m_onetimeInitAction;  
  
    public ObjectPool(int initialBufferSize,  
        Action<T> ResetAction = null,  
        Action<T> OnetimeInitAction = null) {  
  
        m_objectQueue = new Queue<T>(initialBufferSize);  
        m_resetAction = ResetAction;  
        m_onetimeInitAction = OnetimeInitAction;  
    }  
  
    public T New() {  
        if (m_objectQueue.Count > 0) {  
            var t = m_objectQueue.Dequeue();  
  
            m_resetAction?.Invoke(t);  
  
            return t;  
        } else {  
            var t = new T();  
              
            m_onetimeInitAction?.Invoke(t);  
  
            return t;  
        }  
    }  
  
    public void Store(T obj) {  
        m_objectQueue.Enqueue(obj);  
    }  
}
```

+ 此对象池约束 `T` 必须为引用类型，且有一个无参构造函数。
+ 有一个 `reset` 和 `onetimeInit` 的泛型委托，复制重置对象和第一次初始化对象
+ 采用了延迟策略，只有在调用 `New` 和 `Store` 的时候才会初始化或者重置对象


## 被管理类型自重置的池

对于上面那个对象池，我们将其初始化和重置操作和对象本身分离了，造成了紧耦合，因此可以使用一个 `IResetable` 接口，使得类型 `T` 可以由本身初始化或者重置

```csharp
public interface IResetable  
{  
    void Reset();  
    void OneTimeInit();  
}  
  
public sealed class ObjectPool<T>  
    where T: class, IResetable, new()  
{  
    private Queue<T> m_objectQueue;  
  
    public ObjectPool(int initialBufferSize) {  
        m_objectQueue = new Queue<T>(initialBufferSize);  
    }  
  
    public T New() {  
        if (m_objectQueue.Count > 0) {  
            var t = m_objectQueue.Dequeue();  
              
            t.Reset();  
  
            return t;  
        } else {  
            var t = new T();  
              
            t.OneTimeInit();  
  
            return t;  
        }  
    }  
  
    public void Store(T obj) {  
        m_objectQueue.Enqueue(obj);  
    }  
}
```


##  使用 Dictionary 和 List 实现的对象池

```csharp
public sealed class GameObjectPool : MonoBehaviour  
{  
    public static GameObjectPool Instance { get; private set; }  
  
    private Dictionary<string, List<GameObject>> pool = new Dictionary<string, List<GameObject>>();  
  
    private void Awake() {  
        if (Instance == null) {  
            Instance = this;  
        } else {  
            Destroy(gameObject);  
        }  
    }  
  
    public GameObject Get(string prefabName, Vector3 position, Quaternion rotation) {  
        var key = prefabName;  
        GameObject obj;  
          
        // 如果字典里面有这个 key 并且 key 对应的 List 不为空  
        if (pool.ContainsKey(key) && pool[key].Count > 0) {  
            var list = pool[key];  
            obj = list[0];  
            list.RemoveAt(0);  
            // initialize  
            obj.SetActive(true);  
            obj.transform.position = position;  
            obj.transform.rotation = rotation;  
        } else {  
            // 如果对象池里面没找到该对象  
            obj = Instantiate(Resources.Load(prefabName), position, rotation, transform) as GameObject;  
        }  
  
        return obj;  
    }  
  
    public void Store(GameObject obj) {  
	    // 清除 Instantiate 生成的对象名会带有 "(Clone)" 字段
        var key = obj.name.Replace("(Clone)", string.Empty);  
        if (pool.ContainsKey(key)) {  
            pool[key].Add(obj);  
        } else {  
            pool[key] = new List<GameObject>() { obj };  
        }  
        obj.SetActive(false);  
    }  
}
```

此处使用一个 `Dictionary<string, List<GameObject>>` 的数据结构，实现了对任一预制体对象的对象池，提高通用性；每次只需根据对象名字找到相应存储区域即可。