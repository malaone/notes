### selenium

#### selenium-server3

启动命令：

```
java -Dwebdriver.chrome.driver="D:\xk-crawler\chromeDriver\chromedriver.exe" -jar selenium-server-standalone-3.9.1.jar -port 5556
```

默认端口：4444

#### window弹框

示例：

```
安全登录窗口：http://91caj.com/10.1111/pai.12979
用户名密码：91tsgjuhua 778899
```

解决方法：

```java
webDriver.get("http://91tsgjuhua:778899@91caj.com/10.1111/pai.12979");
```

即使不需要用户名密码也可以正常访问：

```java
webDriver.get("http://91tsgjuhua:778899@www.baidu.com");
//等同于
webDriver.get("http://www.baidu.com");
```

其他方案：参考https://blog.csdn.net/qiyueqinglian/article/details/47813331



