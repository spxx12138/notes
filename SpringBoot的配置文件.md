#SpringBoot配置文件加载顺序
下面是spring boot在启动时加载配置文件的部分代码

```java
private String searchLocations;
private Set<String> getSearchLocations() {
    if (this.environment.containsProperty("spring.config.location")) {
    	//如果环境变量配置了spring.config.location,则只查找配置的目录.目录可以配置多个,用逗号分隔开
        return this.getSearchLocations("spring.config.location");
    } else {
    	//如果环境变量中配置了spring.config.additional-location,则将该目录也加入搜索队列.
        Set<String> locations = this.getSearchLocations("spring.config.additional-location");
        locations.addAll(this.asResolvedSet(ConfigFileApplicationListener.this.searchLocations, "classpath:/,classpath:/config/,file:./,file:./config/"));
        return locations;
    }
}

private Set<String> getSearchLocations(String propertyName) {
    Set<String> locations = new LinkedHashSet();
    String path;
    if (this.environment.containsProperty(propertyName)) {
        for(Iterator var3 = this.asResolvedSet(this.environment.getProperty(propertyName), (String)null).iterator(); var3.hasNext(); locations.add(path)) {
            path = (String)var3.next();
            if (!path.contains("$")) {
                path = StringUtils.cleanPath(path);
                if (!ResourceUtils.isUrl(path)) {
                    path = "file:" + path;
                }
            }
        }
    }

    return locations;
}
private Set<String> asResolvedSet(String value, String fallback) {
    List<String> list = Arrays.asList(StringUtils.trimArrayElements(StringUtils.commaDelimitedListToStringArray(value != null ? this.environment.resolvePlaceholders(value) : fallback)));
    // 反转列表中元素的顺序
    Collections.reverse(list);
    return new LinkedHashSet(list);
}
```
默认情况下，会从以下四个地方查找配置文件，优先级为：
	1. 当前目录下config文件夹
	2. 当前目录
	3. classpath目录下的config文件夹
	4. classpath目录 的顺序加载配置文件
- 如果环境变量配置了spring.config.location，则只查找配置的目录。目录可以配置多个，用逗号分隔开，排在最后的优先级最高（因为在asResolvedSet()方法中将集合反转了）
- 如果环境变量中配置了spring.config.additional-location，则将该目录也加入搜索队列。值得注意的是，加载的时候，首先将spring.config.additional-location的配置添加到队列，然后再添加默认配置。所以优先级为spring.config.additional-location配置的倒序、默认的四个目录。
- 当多个配置文件中存在相同配置时，优先级高的配置文件内容会覆盖优先级低的。不同配置可以共存。
- <font color='red'>spring.config.location与spring.config.additional-location是环境变量，在程序启动前加载的，如果配置在application.properties中就太晚了。以jar包形式运行项目时，可以在命令行指定。部署在tomcat中的时候，可以配置在${TOMCAT_HOME}/conf/catalina.properties文件中。</font>

# 多环境下配置文件一键切换
###第一种方法：
1. **spring.profiles.active：**在配置文件的加载目录中（spring.config.location或者spring.config.additional-location、默认的四个位置）创建application-xxx.properties文件。同时在application.properties中添加spring.profiles.active=xxx。可配置多个，用逗号分隔开。
2. **spring.profiles.include：**与spring.profiles.active类似，优先级低于spring.profiles.active。

```
private Set<ConfigFileApplicationListener.Profile> getProfilesActivatedViaProperty() {
	if (!this.environment.containsProperty("spring.profiles.active") && !this.environment.containsProperty("spring.profiles.include")) {
	    return Collections.emptySet();
	} else {
	    Binder binder = Binder.get(this.environment);
	    Set<ConfigFileApplicationListener.Profile> activeProfiles = new LinkedHashSet();
	    activeProfiles.addAll(this.getProfiles(binder, "spring.profiles.include"));
	    activeProfiles.addAll(this.getProfiles(binder, "spring.profiles.active"));
	    return activeProfiles;
	}
}
```
### 第二种方法
在pom文件中配置profile与resource标签，在打包时指定打包哪个路径下的配置文件。
```
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <env>dev</env>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>pro</id>
        <properties>
            <env>pro</env>
        </properties>
    </profile>
    </profiles>
<build>
	<resources>
        <resource>
            <directory>src/main/resources</directory>
            <excludes>
                <exclude>dev/*</exclude>
                <exclude>pro/*</exclude>
            </excludes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/resources/${env}</directory>
            <includes>
                <include>*.*</include>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```