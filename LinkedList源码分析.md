# LinkedList 部分源码分析 #
***jdk1.8***

### 内部类 Node ###
LinkedList使用的是链式结构，将组成链表的节点抽象为内部类Node
```
private static class Node<E> {
	// 该节点保存的数据
    E item;
    // 指向该节点的下个一节点
    Node<E> next;
    // 指向该节点的上个一节点
    Node<E> prev;
	
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```
### 成员变量 ###
通过增加标记节点（头节点、尾节点），可以省去操作时需要判断节点是否为头尾的操作，从而简化代码。
```
// LinkedList的大小
transient int size = 0;
// 头节点
transient Node<E> first;
// 尾节点
transient Node<E> last;
```

### 判断元素出现的位置 ###
```
// 遍历节点，判断节点的item与对象是否相等。空与非空使用两种方式判断。
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}

// 同上，倒循环。
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

### 添加 ###
```
public boolean add(E e) {
	// 默认向尾部添加
    linkLast(e);
    return true;
}
public void add(int index, E element) {
	//检查插入位置是否越界,如果index大于数组的长度，则抛出异常
    checkPositionIndex(index);

    if (index == size)
    	//如果index等于数组的长度，则证明插入位置是在最后一位（包含index与size同为0的情况）
        linkLast(element);
    else
    	//否则 先使用node(index)方法获取到index位置的节点
        linkBefore(element, node(index));
}

//在链表末尾插入一个节点
void linkLast(E e) {
	//将链表的last节点赋给一个常量
    final Node<E> l = last;
    //创建一个节点，该节点的前驱是链表的last节点，该节点值为e，后继为null（因为是在末尾插入的）
    final Node<E> newNode = new Node<>(l, e, null);
    //尾节点为刚才创建的节点
    last = newNode;
    if (l == null)
    	//如果在插入之前l为null，则证明当前的linkedlist是刚初始化的，还没有任何元素，则头头节点与尾节点都是新插入的这个
        first = newNode;
    else
    	//将原last节点的next指向新创建的last节点
        l.next = newNode;
    //数组长度加一
    size++;
    //操作数加一
    modCount++;
}

void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    //数组长度加一
    size++;
    //操作数加一
    modCount++;
}

```
### 删除 ###
```
//删除一个节点，参数为被删除的节点
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
	final Node<E> prev = x.prev;

	// 上一个节点为空，则它是头节点，删除该节点时，头节点指向该节点的下一个节点
    if (prev == null) {
        first = next;
    } else {
    	// 否则该节点上一个节点的下一个节点指向该节点的下一个节点，并且将该节点的上一个节点置为null。
        prev.next = next;
        x.prev = null;
    }
	// 同上
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
	//至此，该节点的两个引用以及保存的元素，全部为null，等待GC回收。
    x.item = null;
    //链表大小减一
    size--;
    //操作数加一
    modCount++;
    return element;
}


// 默认删除第一个，返回被删除的元素
public E remove() {
    return removeFirst();
}

// 删除index处的节点
public E remove(int index) {
	//检查index是否越界
    checkElementIndex(index);
    return unlink(node(index));
}

// 删除指定的对象。只删除一次，即该对象第一次出现的位置。
public boolean remove(Object o) {
	// 根据对象是否为空，需要采用不同的方式来判断相等
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

// 如果没有标记尾节点，就只能一个一个的往下找了
public E removeFirst() {
    final Node<E> f = first;
    // 如果头节点为空则证明表是空的，抛出异常
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

// 删除第一次出现的对象
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

// 删除最后一次出现的对象
public boolean removeLastOccurrence(Object o) {
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

### 查找 ###
```
// 根据下标，使用for循环遍历到index位置，返回Node对象
Node<E> node(int index) {
    // assert isElementIndex(index);

	// 根据index处于链表前半部分还是后半部分，选择从前面遍历还是从后面遍历，以此减少遍历次数。
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

// 返回node的item
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

### 修改 ###
```
// 找到index处的节点，并将其设为新值。
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```