PS：java1.8（1.7没看过。。。）HashMap我就看三个点，这玩意咋构造的？咋保存数据的？咋获取数据的？

### new HashMap到底干了点啥

我一般用hashmap 就是直接new也不带参数

```java
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

貌似就是给loadFactor赋值，这个DEFAULT_LOAD_FACTOR是0.75，loadFactor表示负载因子，那为啥默认值是0.75，官方给的解释是根据各方面考虑给了个0.75，咱也不知道，咱也不敢问

还有几个比较重要的参数：

- threshold：表示临界值，就是说map中的元素个数达到这个值，就要对map进行扩容了，是通过容量和负载因子算出来的（容量 * 负载因子）
- capacity：表示容量（实际上类中并没有这个参数，写出来是为了好理解）默认值是16，这个值表示的就是table的长度
- table：实际存数据的地方（Node<K,V>[]），是一个数组，hashmap被称为散列表是因为这个数组每一个元素都是一个节点，节点下边又可以挂上形成一个List，所以成为散列表
- TREEIFY_THRESHOLD：值为8，表示table中如果一个节点List的数量超过这个值，就会将这个list转换为红黑树
- UNTREEIFY_THRESHOLD：值为6，他与上面那个参数相反，表示如果一个节点是一个红黑树，且数量小于6，就会将这个树转换为list

### put方法干了什么

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
    	// 判断table是否空，为空就初始化table
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
    	// 通过hash值计算出table中对应的index，如果为空直接new节点放在table[index]处
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;// 走到这里就说明table[index]有值了，判断这个key与新key是否相同
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)// 如果不同，再判断是否是树节点
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 循环遍历list，如果找到与新key相同的就break，找不到就new节点并放在list尾部
                // 同时判断是否要转换成红黑树
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                // 新值替换旧值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)// 判断是否需要扩容
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```



上图简单的分析了下代码逻辑，以上有几点需要注意的地方：

1. resize方法被调用了两次，第一次用来被初始化，第二次用来扩容
2. 通过hash值计算index，(n - 1) & hash，n表示table的长度，而默认的长度为16，也就是说index=15 & hash。&是位运算，15的二进制是1111也就是说index的值就等于hash的后四位，这就是table的默认长度为16的原因，同时也说明hash值本身时均匀分布的，hash算法就是均匀的，所以在日常使用时尽量将capacity设置为2的幂
3. 红黑树，能力有限就不分析了，但是这在1.7中是不存在的，是1.8新增的功能
4. 上面讲述到，新的数据会放在list的尾巴处，而1.7中是放在list头部，下面会分析为什么会有这样的改变

### resize是怎么扩容的

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        // 这里删除一些初始的代码
    	// newCap = oldCap * 2
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {	// 循环遍历整个表
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;		// 老表对应位置置空
                    if (e.next == null)	// 如果就一个数据，就直接重新计算index赋值完事
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)	// 红黑树
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // 两个队列，lo表示重新hash后与原先的index相同的数据队列
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 假设oldCap为16,只需要获取hash中倒数第五位的值就好了
                            // 1就表示要迁移了。
                            // 注意：循环是从list头部开始的，循环后的数据也是插入到队列尾部的，
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 将队列放到相应的位置，保证了原先list的顺序不变
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

上面的注释强调两点：1. 区分节点是否要迁移的算法是e.hash & oldCap，如果oldCap的值不是2的幂，那么后果就有点操蛋了，所以这里有点高度依赖capacity是2的幂的意思。2. 扩容后，由于节点采用尾插的方式，所以节点的顺序并没有发生改变，这是与1.7很大的一个差别，1.7采用的头插的方式，在多线程的情况下，很容易造成队列变成一个环。这时候去查询数据就会造成死循环。

### remove是怎么删除数据的

```java
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
    	// 先来一堆判空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            // 判断桶的首个元素是不是就是要删除的
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                // 判断是不是树节点，略过
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {	// 循环遍历，找到对应节点
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)	// 直接将前继节点的next指向当前节点的next结束
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```

