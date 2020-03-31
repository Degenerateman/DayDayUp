### LinkedHashMap

LinkedHashMap继承了HashMap，也就是说，他是在hashmap的基础上加了一下其他功能。

### 构造方法

```java
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }
```

可以看到相比hashmap他就多了一个accessOrder参数，这个参数的意思就是是否需要排序

类中还有两个参数：

```java
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;
```

表示用head和tail维护一个队列。

### put

翻看LinkedHashMap整个类也没有put方法

### get

```java
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```
可以看到，只是调用了hashmap中的方法区获取数据，只是在获取到值后会调用afterNodeAccess这个方法，（accessOrder就构造方法中的参数，这里我们假设为true）