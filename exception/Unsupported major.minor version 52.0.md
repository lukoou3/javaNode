## Unsupported major.minor version 52.0解决办法
项目部署启动时遇到bug
```java
java.lang.UnsupportedClassVersionError: com/algoblu/controller/network/basic/shutdownListener/Shutdown : Unsupported major.minor version 52.0 (unable to load class com.algoblu.controller.network.basic.shutdownListener.Shutdown)
	at org.apache.catalina.loader.WebappClassLoaderBase.findClassInternal(WebappClassLoaderBase.java:3209)
	at org.apache.catalina.loader.WebappClassLoaderBase.findClass(WebappClassLoaderBase.java:1373)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1861)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1735)
	at org.apache.catalina.core.DefaultInstanceManager.loadClass(DefaultInstanceManager.java:495)
	at org.apache.catalina.core.DefaultInstanceManager.loadClassMaybePrivileged(DefaultInstanceManager.java:477)
	at org.apache.catalina.core.DefaultInstanceManager.newInstance(DefaultInstanceManager.java:113)
	at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:5026)
	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5633)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:145)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1700)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1690)
	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:745)
```

使用jdk低版本运行高版本编译的jar会报这种错：

```
报错Unsupported major.minor version 52.0 error

说明代码是用JDK8进行编译的；

报错Unsupported major.minor version 51.0 error

说明代码是用JDK7进行编译的；

报错Unsupported major.minor version 50.0 error

说明代码是用JDK6进行编译的；

报错Unsupported major.minor version 49.0 error
```




