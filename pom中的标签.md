### resources： ###
结构：
```xml
<project>
  <build>
    <resources>
      <resource>
        <directory></directory>
        <includes>
          <include></include>
        </includes>
        <excludes>
          <exclude></exclude>
        </excludes>
        <filtering></filtering>
        <targetPath></targetPath>
      </resource>
    </resources>
  </build>
</project>
```

文档是这样描述的:
> This element describes all of the classpath resources such as properties files associated with a project.These resources are often included in the final package.
> 
>此元素描述所有类路径资源，例如与项目关联的属性文件。这些资源通常包含在最终包中
	
resources可以包含多个resource，resource标签是具体的配置信息。directory描述存储资源的目录，路径相对于POM，默认打包目录下的全部文件以及文件夹，可以使用includes、include、excludes、exclude明确指定包含、排除哪些文件。filtering指定了这些文件是否可以使用占位符代替具体值。


* 当没有配置resource标签时，maven默认打包src/main/resouce文件夹下的全部文件。
* 当配置了一个resource标签时，maven会打包directory指定的目录下的文件。
* 当resource未配置include时，默认打包目录下全部文件。
* 当同一个resource标签内的include与exclude冲突时，那么**以exclude为准**。（经测试，与include和exclude的先后顺序无关）
* 当两个resource标签的include与exclude冲突时，那么**以include为准**。（经测试，与resource的先后顺序无关）
* 还可以使用targetPath标签指定将文件打包到何处，默认是在classes文件夹下。
* filtering可以指定**该resource下**被打包的文件是否使用占位符。关于占位符的使用请看[SpringBoot与Maven占位符](SpringBoot与Maven占位符)


### profiles ###
结构：
```xml
<project>
  <profiles>
    <profile>
      <id></id>
      ...
    </profile>
  </profiles>
</project>
```
文档是这样描述的：
> A listing of project-local build profiles which will modify the build process when activated
> 
> 项目本地构建配置文件的列表，它将在激活时修改构建过程

profile必须定义id！在profile中可以定义大部分在pom中可以定义的标签，包括dependencies、dependencyManagement、build等等，当profile被激活时，它里面定义的内容会覆盖与pom中相同的内容。
<br>最常用的几个标签有：
- id：这个是必须定义的，profile的唯一标识
- properties：这个标签可以定义在整个POM中用作替换的属性，主要用来定义在不同环境下的变量。
- activation：定义该profile在什么情况下生效
  - activeByDefault：true/false，是否设为默认
  - file：它有两个子标签exists与missing，将根据文件是否存在激活此配置文件，如果配置多个，则他们之间的关系是‘与’，即都满足才激活。
  - jdk：根据jdk的版本来决定是否激活。可以使用！表示非，如1.8，！1.8。
  - os：根据操作系统来决定是否激活。
  - property：根据配置的properties，filter属性激活此配置文件。