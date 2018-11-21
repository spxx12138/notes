# ArrayList 部分源码分析 #
***jdk1.8***

### 成员变量 ###
```java
// 默认大小
private static final int DEFAULT_CAPACITY = 10;

// 保存元素的数组
transient Object[] elementData;

// 保存的元素个数，不等于elementData的长度！
private int size;

// 两个空数组
private static final Object[] EMPTY_ELEMENTDATA = {};
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

```

### 初始化 ###
将elementData初始成参数大小的数组，无参或参数为0则为一个空数组。
```java
// 在添加前就已知元素个数，即使不知道也可以估计一个大概值，避免多次扩容。
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

### 去除数组空余位置 ###
数组在动态扩容过程中可能会导致数组长度远远大于元素个数。
当不会继续向数组中添加元素时，可以使用该方法去除空余位置以节省空间。
其实就是创建一个新的与元素个数相同的数组然后赋值。
```java
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```

### 扩容 ###
```java
// 手动扩容，在添加的过程中才确定所有元素的个数，可以使用该方法一次扩容到所需大小。
public void ensureCapacity(int minCapacity) {
	// 如果elementData是空数组，最小长度为默认长度
	// 如果elementData不是空数组，允许的最小长度为0
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)  ? 0 : DEFAULT_CAPACITY;
	
	// 扩容长度不能小于最小长度
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

// 私有扩容方法
private void ensureCapacityInternal(int minCapacity) {
	// 如果elementData是空数组，扩容的最小长度为默认长度与参数两者间最大的
	// 也就是当elementData长度为0时，扩容的最小尺寸不能小于10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
	
	// 判断是否需要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // 每次扩容，新容量不应小于原容量的1.5倍。
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
        
    // 如果新长度大于所允许的最大长度，则新长度为ArrayList所允许的最大长度
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 将原数组拷贝到新数组（调用的是System.arraycopy方法）
    elementData = Arrays.copyOf(elementData, newCapacity);
}

// 有些虚拟机会在头部保存一些信息，设为最大值减八可以减少出错几率
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0)
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

### 判断元素位置 ###
```java
// 根据元素出现位置判断是否包含
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

// 遍历数组判断元素是否与需要查找的对象相等，空与非空需要使用不同的判断方式。未找到则返回负一
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
// 同上，只不过是倒序查找。
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

### 添加 ###
```java
public boolean add(E e) {
	// 判断添加一个元素后所需的大小是否需要扩容
    ensureCapacityInternal(size + 1);
    // 将对象放到下一个位置
    elementData[size++] = e;
    return true;
}

public void add(int index, E element) {
    rangeCheckForAdd(index);
	// 判断添加元素后是否需要扩容
    ensureCapacityInternal(size + 1);
    // 将index处以后的所有元素复制到index+1之后
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // index处为新值
    elementData[index] = element;
    size++;
}
```

### 删除 ###
```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

	// 需要移动元素的个数，为0则是最后一个元素
    int numMoved = size - index - 1;
    if (numMoved > 0)
    	// 删除的不是最后一个元素，后续元素前移。
    	// 由于复制目标是原数组，最后一位没有被覆盖，导致数组仍为原长，后两个元素重复。
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 最后一个元素置为null
    elementData[--size] = null; 
    
    return oldValue;
}

// 遍历判断是否相等然后删除，只删除第一个出现的，空与非空用不同方法判断。
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// 检查下标是否越界
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```
### 查找 ###
```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```
### 修改 ###
```java
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```