# Maven的占位符机制 #
maven在打包的时候可以指定resource资源目录，具体请参见另一篇文章[pom中的标签](pom中的标签)。

resource下的filtering标签如果配置为true时，maven在打包时会读取被打包的文件，并将其中的占位符替换为实际值。filtering只对当前的resource生效。

maven的占位符分为以下几类：
1. env.X：操作系统环境变量，引用方式${env.JAVA_HOME}
2. project.x：pom文件中的属性，如：<project><version>1.0</version></project>，引用方式：${project.version}
3. settings.x：settings.xml文件中的属性，如：<settings><offline>false</offline></settings>，引用方式：${settings.offline}
4. Java System Properties：java.lang.System.getProperties()中的属性，比如java.home，引用方式：${java.home}
5. 自定义：在pom文件中可以：<properties><installDir>c:/apps/cargo-installs</installDir></properties>，引用方式：${installDir}

maven默认的占位符为${*}，可以自定义，自定义方式如下
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-resources-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <useDefaultDelimiters>false</useDefaultDelimiters>
    <delimiters>
    <delimiter>$[*]</delimiter>
    </delimiters>
    <encoding>UTF-8</encoding>
  </configuration>
</plugin>
```

# 在SpringBoot中使用Maven占位符 #
占位符含义与类别都是相同的，不过由于SpringBoot中默认使用的maven插件为:
```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```
所以mavne的占位符格式为@ * @，且不支持自定义。这是因为${ * }为SpringBoot的占位符,两者之间冲突。

假设application.properties被maven过滤。打包前文件内容为
```properties
p1=${JAVA_HOME}
p2=@JAVA_HOME@
```
打包后文件内容为
```properies
p1=${JAVA_HOME}
p2=/usr/local/java/jdk1.8.0_131
```
在程序中使用p1，值也是/usr/local/java/jdk1.8.0_131。
- p1使用的是SpringBoot占位符，打包时不会转化为实际值，在SpringBoot程序运行时才读取环境变量到内存。
- p2使用的是maven占位符，在maven打包的时候，就被读取转换为实际值了。
