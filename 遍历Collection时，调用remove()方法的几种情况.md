# 遍历Collection时，调用remove()方法的几种情况 #

****<font color='red'>结论：推荐在迭代器中使用迭代器的remove()方法删除元素，for循环容易造成下标混乱，foeach中删除会报错</font>****

遍历有三种方式：
- for循环（while同）
- foreach
- 迭代器

让我们分别看一下当在这三种循环过程中，调用remove()会发生什么。
首先创建一个ArrayList，里面存储由1至10的正整数。
```
ArrayList<Integer> list = new ArrayList<>();
for (int i = 0; i < 10; i++) {
    list.add(i);
}
```
- for循环过程中删除

	```
	for (int i = 0; i < list.size(); i++) {
		list.remove(i);
	}
	System.out.println(list.size());
	```
**该方法并没有将数组中的每一个元素都删除掉**，此时数组长度为5，数组中的元素有 1 3 5 7 9；因为删除了第i个元素，i之后的元素向前移一位，所以出现了这种间隔式的删除，同时 删除了一个元素后，list的size会随之改变，所以不会出现下标越界的情况。
*每调用一次remove()，就i--，即可将所有元素都遍历到，但容易造成混淆*
- foreach过程中删除

	```
	for (int i : list) {
		list.remove(i);
	}
	```
	抛出<font color='red'>ConcurrentModificationException</font>异常
*remove()是个重载方法，可传入int型的数组下标也可传入Object对象。曾经有过这样的疑问，当数组里的内容是int型时，那么删除的是哪个呢？这取决于参数的类型，如果传入的是‘3’，那么它属于基本类型int型，删除的是下标。如果使用封装类型封装一下：Integer.valueof(3),那么它是一个对象是Object类型，删除的是‘3’这个元素。*
**集合类型的foreach方法，本质上使用的是集合的迭代器**，将这段代码编译成class文件再反编译回java文件，结果如下：
```
ArrayList localArrayList = new ArrayList();
    for (int i = 0; i < 10; i++) {
	localArrayList.add(Integer.valueOf(i));
}
for (Iterator localIterator = localArrayList.iterator(); localIterator.hasNext(); ) { 
	int j = ((Integer)	localIterator.next()).intValue();
	localArrayList.remove(j);
    System.out.println(localArrayList.get(j));
}
```
*数组类型的foreach等于普通的for循环*
- 迭代器过程中删除
	1. 使用集合的remove()方法删除

	```
	Iterator<Integer> iter = list.iterator();
	while (inter.hasNext()) {
	    list.remove(inter.next());
	}
	```
抛出<font color='red'>ConcurrentModificationException</font>异常
	2. 使用迭代器的remove()方法删除	
	```
	Iterator<Integer> inter = list.iterator();
    while (inter.hasNext()) {
        inter.next();
        inter.remove();
    }
    System.out.println(list.size());
	```
正常删除，最后输出数组长度为0。


# 迭代器与modCount #

### modCount ###
modCount用来记录修改次数，出现在线程不安全的集合类中。每当集合有影响到元素个数的操作时，操作数加一。（set方法虽然修改了元素但是不改变元素个数，不会使modCount增加）
主要用于迭代过程中判断集合类是否被外部修改。
 
### 迭代器 ###
每个集合的内部结构不同，如ArrayList维护的是一个数组，LinkedList采用的是链表结构。为了使遍历元素的过程中不用考虑数据结构的内部实现，Java采用迭代器为各种数据机构提供一套相同的操作标准。
Iterable是一个接口，实现了该接口的类需要返回一个Iterator迭代器。Iterator同样是一个接口，其中包含了hasNext(),next(),remove()几个方法。
Collection实现了Iterable接口，所以List，Set的各种实现方式都可以使用迭代器进行遍历。
```
// 迭代器初始化时，使用expectedModCount记录modCount当前操作数
int expectedModCount = modCount;

// 迭代器的大部分方法如add，remove等，在第一行都会调用该方法判断集合是否被修改过。
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```
<font color='red'>使用迭代器遍历集合过程中若使用集合自身的add，remove等方法修改集合元素的个数导致modCount发生变化，那么迭代器在下次操作集合的时候就会发现modCount与expectedModCount不同，进而抛出异常。而迭代器的remove虽然调用的也是集合自身的remove，但是在调用结束后会将modCount与expectedModCount同步，所以不会抛出异常。</font>
