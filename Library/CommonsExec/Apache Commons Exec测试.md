## Apache Commons Exec测试

用到的python脚本：
```py
import time

if __name__ == '__main__':
    for i in range(1, 1000):
        time.sleep(0.1)
        print("十次方是发多少多线程是受持读诵查的是订单生产三大城市的十点多" + str(i) * 5)
```

### 基本功能的测试
```java
import org.apache.commons.exec.*;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.IOException;

public class ExecTest {

    public static void main(String[] args) throws Exception{
        exec();
        //exec1();
        //execAsync();
    }

    public static void execAsync() throws IOException {
        System.out.println(new File(".").getAbsolutePath());
        String line = "python pyutils/test/out.py";
        CommandLine cmdLine = CommandLine.parse(line);

        DefaultExecutor executor = new DefaultExecutor();

        //ByteArrayOutputStream的数组缓冲区会自动扩容
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();//接收正常结果流
        ByteArrayOutputStream errorStream = new ByteArrayOutputStream();//接收异常结果流
        PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream,errorStream);
        executor.setStreamHandler(streamHandler);

        ExecuteWatchdog watchdog = new ExecuteWatchdog(60*1000);//设置超时
        executor.setWatchdog(watchdog);

        DefaultExecuteResultHandler handler = new DefaultExecuteResultHandler();

        int exitValue = 0;
        try {
            executor.execute(cmdLine,handler);

            while (!handler.hasResult()){
                handler.waitFor(1000);
                String out = outputStream.toString("gbk");
                String error = errorStream.toString("gbk");
                System.out.println(out);
                System.out.println("-----------------------------------------");
            }

            String out = outputStream.toString("gbk");
            String error = errorStream.toString("gbk");
            System.out.println(out);

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            outputStream.close();
            errorStream.close();
        }


        System.out.println(exitValue);
    }

    public static void exec1() throws IOException {
        System.out.println(new File(".").getAbsolutePath());
        String line = "python pyutils/test/out.py";
        CommandLine cmdLine = CommandLine.parse(line);

        DefaultExecutor executor = new DefaultExecutor();

        //ByteArrayOutputStream的数组缓冲区会自动扩容
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();//接收正常结果流
        ByteArrayOutputStream errorStream = new ByteArrayOutputStream();//接收异常结果流
        PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream,errorStream);
        executor.setStreamHandler(streamHandler);

        ExecuteWatchdog watchdog = new ExecuteWatchdog(6*1000);//设置超时，超时会抛出异常
        executor.setWatchdog(watchdog);

        int exitValue = 0;
        try {
            exitValue = executor.execute(cmdLine);//超时会抛出异常
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            outputStream.close();
            errorStream.close();
        }
        //不同操作系统注意编码，否则结果乱码
        String out = outputStream.toString("gbk");
        String error = errorStream.toString("gbk");
        System.out.println(out);
        System.out.println(exitValue);
    }

    public static void exec() throws IOException {
        String line = "ping 127.0.0.1";
        CommandLine cmdLine = CommandLine.parse(line);
        DefaultExecutor executor = new DefaultExecutor();

        /**
         * 关于这个exitValue：
         一般的进程，其运行结束后，默认值为0。
         对于某些有特殊退出值的程序，需要确定知道其值是什么，然后用executor.setExitValue(退出值);进行明确设置，否则，会出现ExecuteException。
         比如，写一个JAVA程序，在main函数中，用System.exit(10);退出程序，用Exec管理这个进程就必须设置：executor.setExitValue(10);
         如果程序有多个退出值，可使用executor.setExitValues(int[]);函数进行处理。
         */
        executor.setExitValue(0);

        int exitValue = executor.execute(cmdLine);
        System.out.println(exitValue);
    }
    
}
```

### 输出日志到控制台并设置编码
继承LogOutputStream容易实现，由于java中没有属性的重写，并且属性的访问不像方法有多态，所以用了反射的方式获取父类的属性，在构造方法中只需获取一次，效率高一点：
```java
import org.apache.commons.exec.LogOutputStream;

import java.io.ByteArrayOutputStream;
import java.io.UnsupportedEncodingException;
import java.lang.reflect.Field;

public class ExecPrintStream extends LogOutputStream {
    private ByteArrayOutputStream buffer;
    private String charsetName;

    public ExecPrintStream(String charsetName) {
        super();
        try {
            //java中没有属性的覆写，scala中有(其实就是通过方法实现的)
            Field bufferField = LogOutputStream.class.getDeclaredField("buffer");
            bufferField.setAccessible(true);
            buffer = (ByteArrayOutputStream) bufferField.get(this);
        } catch (Exception e) {
        }
        this.charsetName = charsetName;
    }

    @Override
    protected void processBuffer() {
        try {
            processLine(buffer.toString(charsetName));
            buffer.reset();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void processLine(String line, int logLevel) {
        System.out.println(line);
    }
}
```

测试代码：
```java
import org.apache.commons.exec.CommandLine;
import org.apache.commons.exec.DefaultExecutor;
import org.apache.commons.exec.ExecuteWatchdog;
import org.apache.commons.exec.PumpStreamHandler;

import java.io.File;
import java.io.IOException;

public class ExecStreamTest {

    public static void main(String[] args) throws Exception{
        exec();
    }

    public static void exec() throws IOException {
        System.out.println(new File(".").getAbsolutePath());
        String line = "python pyutils/test/out.py";
        CommandLine cmdLine = CommandLine.parse(line);

        DefaultExecutor executor = new DefaultExecutor();

        ExecPrintStream printStream = new ExecPrintStream("gbk");
        PumpStreamHandler streamHandler = new PumpStreamHandler(printStream);
        executor.setStreamHandler(streamHandler);

        ExecuteWatchdog watchdog = new ExecuteWatchdog(60*1000);//设置超时，超时会抛出异常
        executor.setWatchdog(watchdog);

        int exitValue = 0;
        try {
            exitValue = executor.execute(cmdLine);//超时会抛出异常
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            printStream.close();
        }
        System.out.println(exitValue);
    }
}
```


