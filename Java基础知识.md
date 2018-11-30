### 1. count++ 与 ++count ###
count++是先取值，再进行++操作，++count则是先进行++再取值。

这样理解很容易出错，如：
```java
int count = 0;
int num = count++;
System.out.println(num);
System.out.println(count);
```
输出结果：0 1

这种情况下，按照num = count; count = count+1;这样理解是没错的。但是：
```java
int count = 0;
count = count++;
System.out.println(count);
```
输出结果：0。

并没有按照想象中的：count = count;count = count + 1;这样执行。

这是因为上述代码编译后：
```java
int count = 0;
byte var10000 = count;
int var2 = count + 1;
count = var10000;
System.out.println(count);
```
所以第一个代码块应该这样理解：**在执行num = count++的时候，会先将count赋给一个临时变量，然后执行count=count+1，最后再将临时变量赋给num**
