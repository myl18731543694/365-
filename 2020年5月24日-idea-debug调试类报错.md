## idea debug调试类报错

**测试类**

```java
public class Test {
    public static void main(String[] args) {
        System.out.println("测试");
    }
}
```

**当idea，debug这个类时。报错信息**

```
Connected to the target VM, address: '127.0.0.1:51525', transport: 'socket'
Unexpected error (103) returned by AddToSystemClassLoaderSearch
Unable to add C:\Users\������\AppData\Local\JetBrains\IdeaIC2020.1\captureAgent\debugger-agent.jar to system class path - the system class loader does not define the appendToClassPathForInstrumentation method or the method failed
FATAL ERROR in native method: processing of -javaagent failed, appending to system class path failed
Disconnected from the target VM, address: '127.0.0.1:51525', transport: 'socket'

Process finished with exit code 1
```

**出现原因**

经网上了解，说是因为中文问题导致的debug模式失效。

**解决方案**

- File -> Settings -> Build, Execution, Deployment -> Debugger -> Async Stack Traces
- 取消勾选 instrumenting agent

- **至于为什么要取消这个就可以运行了，网上暂时未查到资料**