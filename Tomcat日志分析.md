# Tomcat日志分析 #

### 一、基本信息 ###
* <font color='red'>**%a - 远程IP地址**</font>
* **%A - 本地IP地址**
* **%b - 发送的字节数：**不包括HTTP头，如果没有发送字节返回“ - ”。</font>
* <font color='red'>**%B - 发送的字节数：**不包括HTTP头，如果没有发送字节返回“ 0 ”。</font>
* **%h - 远程主机名:**如果设置resolveHost=false，则记录的是远程IP地址，但是将resolveHost设为true记录的仍为IP地址，不知道为什么。
* <font color='red'>**%H - 请求协议**</font>
* **%l (小写的L)：**远程逻辑从identd的用户名（总是返回' - '），那么它存在的意义？
* <font color='red'>**%m - 请求方法**</font>
* **%p - 本地端口：**Tomcat的端口还用记录么？
* **%q - 查询字符串：**GET请求' ? '后边的内容，包括'?'。
* <font color='red'>**%r - HTTP请求第一行：**请求方法、URI、协议/版本，相当于 %m **%U%q** %H 一起使用</font>
* <font color='red'>**%s - 响应的HTTP状态码**</font>
* **%S - Session的ID**
* <font color='red'>**%t - 日期和时间**</font>
* **%u - 远程用户身份验证：**指的是登录Tomcat的用户，不是你部署在Tomcat中的应用的用户。
* <font color='red'>**%U - 请求的URL路径：**</font>不会记录GET请求的查询字符串，配合%q使用效果会好一点。
* **%v - 本地服务器名：**记录的还是本机的IP
* <font color='red'>**%D - 处理请求的时间（以毫秒为单位）**</font>
* **%T - 处理请求的时间（以秒为单位）**
* **%I （大写的i） - 当前请求的线程名称**

### 二、详细信息 ###
- **%{XXX}i xxx是请求头属性名(HTTP Request)：** *如 %{User-Agent}i*
- **%{XXX}o xxx是响应头属性名(Http Resonse)**
- **%{XXX}c xxx代表特定的Cookie名**
- **%{XXX}r xxx代表ServletRequest属性名**
- **%{XXX}s xxx代表HttpSession中的属性名**

---
<br>
在分析日志文件时，可以使用这种方式转换时间。

```java
	String viewtime = "21/Sep/2017:15:43:29 +0800";
	SimpleDateFormat sdf =new SimpleDateFormat("dd/MMM/yyyy:HH:mm:ss Z", Locale.US);
	Date time = sdf.parse(viewtime);
```