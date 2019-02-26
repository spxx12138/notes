# 在Bean中使用枚举类型属性 #

在设计数据表时，有一种常量字段是不需要动态扩展的但是数量又很多，用数字来表示在写代码时很难分得清每个数字代表什么。
这个时候就可以用枚举来表示。

先定义一个枚举：
```
public enum MyEnum {

    WORK(1, "事业"), MARRIAGE(2, "婚姻");

    private int key;
    private String name;

    MyEnum(int key, String name) {
        this.key = key;
        this.name = name;
    }

    public int getKey() {
        return this.key;
    }

    public String getName() {
        return this.name;
    }

    @Override
    public String toString() {
        return getName();
    }
    
    public static MyEnum fromKey(int key) {
        Objects.requireNonNull(value, "value can not be null");
        MyEnum myEnum = null;
        for (MyEnum e : MyEnum.values()) {
            if (key == e.getKey()) {
                myEnum = e;
            }
        }
        return myEnum;
    }

}
```
再定义一个bean（略去一些注解和GetSet）：
```
public class MyTable{

	private String id;
	
	private MyEnum myEnum;
	
}

```
这样的bean保存到数据库枚举类型对应的字段保存的是数字0、1、2、3，具体是什么取决于枚举类型出现的顺序。如work保存到数据库就是0，marriage就是1。从数据库取出来的时候会自动转换成枚举类型。可以加上注解使保存到数据库的是枚举的名字。
```
    @Enumerated(EnumType.STRING)
    private MyEnum myEnum;
```
EnumType还有一个选项是ORDINAL，就是默认的保存数字。
<font color='red'>这样做有一个缺点是一旦枚举的顺序被打乱，那整个数据库就乱套了，所以我们要改进一下，自定义一个转换器</font>

```
@Converter
public class EnumConverter implements AttributeConverter<MyEnum, Integer> {

	//转化为数据库的字段，插入保存时需要
    @Override
    public Integer convertToDatabaseColumn(MyEnum myEnum) {
        return myEnum.getKey();
    }

	//将数据库的字段转化为bean属性，查询时需要
    @Override
    public PrayEnum convertToEntityAttribute(Integer dbData) {
        return MyEnum.fromKey(dbData);
    }
}
```

在bean属性上加上注解
```
	@Convert(converter = EnumConverter.class)
    private MyEnum myEnum;
```

大功告成
