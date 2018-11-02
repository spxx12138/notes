# ArrayList与LinkedList 的增删、访问 速度对比实验 #

ArrayList基于数组实现，随机访问快，增删慢。
LinkedList是链表结构，增删快，随机访问慢。
**<font color = 'red'>这是网络与书籍里随处可见的两句话，但并不是说所有情况下Linked的增删都比ArrayList快</font>**


## 随机插入实验 ##
```
public static void linkedTest() {
    LinkedList<Integer> linkedList = new LinkedList<>();
    for (int i = 0; i < 100000; i++) {
        linkedList.add((int) (Math.random() * linkedList.size()), 1);
    }
}
public static void arrayTest() {
    ArrayList<Integer> linkedList = new ArrayList<>();
    for (int i = 0; i < 100000; i++) {
        linkedList.add((int) (Math.random() * linkedList.size()), 1);
    }
}
```
LinkedList 随机插入十万数据用时：15738（15秒）
ArrayList 随机插入十万数据用时：324（0.3秒）

LinkedList 随机插入百万数据用时：8423510（两个半小时）
ArrayList 随机插入百万数据用时：43300（43秒）

**LinkedList在每一次插入之前，内部都会使用for循环找到需要插入的位置，这个代价是很大的。ArrayList虽然会将插入位置之后的元素依次后移，但比LinkedList的访问要快。**
## 末尾插入实验 ##
```
public static void linkedTest() {
    LinkedList<Integer> linkedList = new LinkedList<>();
    for (int i = 0; i < 10000000; i++) {
        linkedList.addLast((int) (Math.random() * 1000));
    }
}

public static void arrayTest() {
    ArrayList<Integer> arrayList = new ArrayList<>();
    for (int i = 0; i < 10000000; i++) {
        arrayList.add(i, (int) (Math.random() * 1000));
    }
}
```
LinkedList尾部插入一百万数据用时：132（零点一秒）
ArrayList尾部插入一百万数据用时：55（零点零五秒）

LinkedList尾部插入一千万数据用时：8994（九秒）
ArrayList尾部插入一千万数据用时：2702（二点七秒）

*<font color='red'>这里有一个疑问，arraylist会自动扩容，那为什么设置了初始长度之后，所用时间反而比没设置、需要多次扩容的用时还长呢</font>*

**LinkedList在插入的时候，除了需要创建新节点，还需要更新原来节点的next，使它指向新插入的节点，所以说 即使是在末尾插入并且ArrayList需要动态扩容的情况下，LinkedList速度仍不及ArrayList。**

## 头部插入实验 ##
```
public static void linkedTest() {
    LinkedList<Integer> linkedList = new LinkedList<>();
    for (int i = 0; i < 1000000; i++) {
        linkedList.addFirst((int) (Math.random() * 1000));
    }
}

public static void arrayTest() {
    ArrayList<Integer> arrayList = new ArrayList<>();
    for (int i = 0; i < 1000000; i++) {
        arrayList.add(0, (int) (Math.random() * 1000));
    }
}
```
LinkedList尾部插入一百万数据用时：131（零点一秒）
ArrayList尾部插入一百万数据用时：129945（两分钟）

**ArrayList在头部插入，每一次插入都会将后面的所有元素后移一位，效率非常低。LinkedList在数据量相同时，因为是双向链表且不需要移动其他元素，无论在头部还是尾部插入，耗时是一样的。**

## 随机删除 ##
```
public static void linkedTest() {
    LinkedList<Integer> linkedList = new LinkedList<>();
    //插入一百万数据
    for (int i = 0; i < 1000000; i++) {
        linkedList.addLast((int) (Math.random() * 1000));
    }
    Date start = new Date();
    //随机删除一万个，占总容量百分之一
    for (int i = 0; i < 00000; i++) {
        linkedList.remove((int)(Math.random()*100000));
    }
    Date end = new Date();
    System.out.println("LinkedList随机删除用时：" + (end.getTime() - start.getTime()));
}

public static void arrayTest() {
    ArrayList<Integer> arrayList = new ArrayList<>();
    //插入一百万数据
    for (int i = 0; i < 1000000; i++) {
        arrayList.add(arrayList.size(), (int) (Math.random() * 1000));
    }
    Date start = new Date();
    //随机删除一万个，占总容量百分之一
    for (int i = 0; i < 10000; i++) {
        arrayList.remove((int)(Math.random()*100000));
    }
    Date end = new Date();
    System.out.println("ArrayList随机删除用时：" + (end.getTime() - start.getTime()));
}
```
ArrayList随机删除用时：5879（六秒）
LinkedList随机删除用时：3752（四秒）
*一千万数据没试，ArrayList删除太慢了，明显高于LinkedList*
*<font color='red'>值得一提的是，如果remove传入的不是下标而是对象类型，那么两种类型的list删除都会更慢，ArrayList：18931
LinkedList：79321</font>*
**虽然LinkedList每次删除时都需要获取到删除的位置，但相比于ArrayList的移动元素，仍然是省时的。**



## 遍历删除 ##
```
public static void linkedTest() {
    LinkedList<Integer> linkedList = new LinkedList<>();
    //插入一百万数据
    for (int i = 0; i < 1000000; i++) {
        linkedList.addLast((int) (Math.random() * 1000));
    }
    Date start = new Date();
    Iterator<Integer> iter = linkedList.iterator();
    while (iter.hasNext()) {
        if ((iter.next() % 2) == 0) {
        	//删除所有偶数
            iter.remove();
        }
    }
    Date end = new Date();
    System.out.println("LinkedList删除用时：" + (end.getTime() - start.getTime()));
}

public static void arrayTest() {
    ArrayList<Integer> arrayList = new ArrayList<>();
    //插入一百万数据
    for (int i = 0; i < 1000000; i++) {
        arrayList.add(arrayList.size(), (int) (Math.random() * 1000));
    }
    Date start = new Date();
    for(int i =0;i<arrayList.size();i++){
        if(arrayList.get(i)%2 == 0){
        	//删除所有偶数
            arrayList.remove(i);
        }
    }
    Date end = new Date();
    System.out.println("ArrayList删除用时：" + (end.getTime() - start.getTime()));
}
```
*为了防止每次随机插入的数据奇偶数的数目差距过大，所以测试了很多次，耗时基本一致。ArrayList用迭代器遍历更慢*
ArrayList遍历删除用时：39235（四十秒）
LinkedList遍历删除用时：16（零点零二秒）
**很明显，LinkedList在删除时不需要移动其他元素的优势在此就体现出来了，因为是使用迭代器遍历，在删除时不需要再次寻找该元素，所以速度很快。**
<font color='red'>LinkedList的遍历千万不要用for循环get，LinkedList的get方法内部是通过循环找到第i个元素的，这样相当于使用了嵌套循环</font>

**需要频繁删除或者头部插入的时候，使用LinkedList
在尾部插入，不怎么进行删除操作时，使用ArrayList**