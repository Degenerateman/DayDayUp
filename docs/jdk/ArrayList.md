ArrayList存储的数据结构是一个数组，默认的大小为10

```java
    Object[] elementData;
	Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;//
    }
```

初始化时会赋一个长度为0的数组

```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;	// 在数组中新增值
        return true;
    }
    private void ensureCapacityInternal(int minCapacity) {
        // 第一次添加元素是是空的，比较默认大小和指定的大小去最大值=10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;// 版本号

        // overflow-conscious code
        // 这里并没有像map一样有一个负载因子，就是直接判断elementData够不够用
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);// 扩容
    }
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 新的数组大小为原数组大小的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

上面有两点需要注意的地方，数组会在容量满的情况下才会扩容，扩容后的大小是原来的1.5倍

```java
    public E remove(int index) {
        rangeCheck(index);// 检测index的值时候越界

        modCount++;
        E oldValue = elementData(index);// 获取老的数据

        int numMoved = size - index - 1;// 需要迁移的数据个数
        if (numMoved > 0)
            // 迁移数据
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        // 将最后一个数据清除掉
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

以上可以看到ArrayList在删除数据的时候需要进行数据迁移，效率很低，而获取数据时是直接获取值不需要遍历数组，所以Array更适合查询多的场景