## servlet生命周期

#### 1. 加载和实例化

Servlet容器负责加载和实例化Servlet。当servlet容器启动时，或者servlet需要执行第一个请求时，先会实例化一个servlet对象



#### 2. 初始化servlet

当实例化servlet这个对象后，servlet容器会调用servlet的init()这个方法，init()方法只会被调用一次



#### 3. 处理请求

当收到请求时，servlet容器调用servlet实例的service()方法



#### 4. 服务终止

当某个servlet实例需要移除时，容器会调用destroy()方法，释放资源