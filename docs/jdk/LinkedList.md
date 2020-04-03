LinkedList存储的数据结构是一个队列

```java
	transient int size = 0;

    transient Node<E> first;

    transient Node<E> last;

    public LinkedList() {
    }
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

```java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    void linkLast(E e) {
        final Node<E> l = last;
        // 构建一个node
        final Node<E> newNode = new Node<>(l, e, null);
        // 将last指向新的节点
        last = newNode;
        // 如果l是null说明是空队列，将都指针指向当前节点
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;//版本
    }
```

当然还有addFirst的方法，只不过大意与上面类似

```java
    public E get(int index) {
        checkElementIndex(index);// 检测index是否越界
        return node(index).item;
    }
    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
    
    Node<E> node(int index) {
        // assert isElementIndex(index);
		// 这里对查询进行了优化，先判断index在整个队列的前半部分还是后半部分，前半部分就从头节点开始查询，后半部分就从尾节点开始查询
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

```java
    public E remove(int index) {
        checkElementIndex(index);// 检测index是否越界
        return unlink(node(index));// 获取节点
    }
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
		// 帮助GC
        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

以上可以看出，LinkedList在删除节点和新增节点的时候很快，不需要像ArrayList一样迁移数据，但是在查询的时候需要遍历队列，所以更适合增删多的场景