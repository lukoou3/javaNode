## Apache Commons Exec
Java管理进程，API级别是使用：Runtime.getRuntime().exec(“shell”);这个方法。 

你可能会认为java自身的Runtime.exec()很简单，而Apache Commons Exec太过臃肿，纯粹是在浪费时间。

但是，我在使用Runtime.exec()的过程时，经历了一系列痛苦的过程。一起来看下commons exec是怎么把这一过程变简单的。

Java在执行命令时输出到某个Buffer里，这个Buffer是有容量限制的，如果满了一直没读取，就会一直等待，造成进程锁死的现象。 

使用Apache Commons Exec，应该可以避免很多类似的坑。 

它提供一些常用的方法用来执行外部进程，另外，它提供了监视狗Watchdog来设监视进程的执行超时，同时也还实现了同步和异步功能， 

Apache Commons Exec涉及到多线程，比如新启动一个进程，Java中需要再开三个线程来处理进程的三个数据流，分别是标准输入，标准输出和错误输出。

扩展：Java进程Runtime、Process、ProcessBuilder调用外部程序
https://blog.csdn.net/c315838651/article/details/72085739
### 引入依赖
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-exec</artifactId>
    <version>1.3</version>
</dependency>
```

### 基本用法
就三步：   
1）创建命令行：CommandLine   
2）创建执行器：DefaultExecutor   
3）用执行器执行命令：executor.execute(cmdLine);  

```java
String cmdStr = "ping www.baidu.com -t";
final CommandLine cmdLine = CommandLine.parse(cmdStr);
DefaultExecutor executor = new DefaultExecutor();
int exitValue = executor.execute(cmdLine);
```

#### 关于exitValue
关于这个exitValue：   

一般的进程，其运行结束后，默认值为0。   
对于某些有特殊退出值的程序，需要确定知道其值是什么，然后用executor.setExitValue(退出值);进行明确设置，否则，会出现ExecuteException。   
比如，写一个JAVA程序，在main函数中，用System.exit(10);退出程序，用Exec管理这个进程就必须设置：executor.setExitValue(10);   

如果程序有多个退出值，可使用executor.setExitValues(int[]);函数进行处理。  

#### 通过添加参数方式构建命令
通过添加参数方式构建命令。这是官方推荐的方式！ 

上面的程序可以改为：
```java
final CommandLine cmdLine = new CommandLine("ping");
cmdLine.addArgument("www.baidu.com");
cmdLine.addArgument("-t");
DefaultExecutor executor = new DefaultExecutor();
int exitValue = executor.execute(cmdLine);
```

### 超时管理
设置外部命令执行等待时间，如果超过设置的等待时间，则中断执行。默认超时会抛出异常。
```java
final CommandLine cmdLine = CommandLine.parse("ping www.baidu.com -t");
ExecuteWatchdog watchdog = new ExecuteWatchdog(5000);//设置超时时间：5秒
DefaultExecutor executor = new DefaultExecutor();
executor.setWatchdog(watchdog);
executor.setExitValue(1);//由于ping被到时间终止，所以其默认退出值已经不是0，而是1，所以要设置它
int exitValue = executor.execute(cmdLine);
```

### 非阻塞方式执行进程
上面的执行外部命令都是阻塞式，也就是在执行外部命令时，当前线程是阻塞的。    
比如执行ping -t，当成的JAVA程序会一直等着不会停止。    
解决办法：使用DefaultExecuteResultHandler处理外部命令执行的结果，释放当前线程。    
```java
final CommandLine cmdLine = CommandLine.parse("ping www.baidu.com -t");
final DefaultExecuteResultHandler resultHandler = new DefaultExecuteResultHandler();

DefaultExecutor executor = new DefaultExecutor();
executor.execute(cmdLine, resultHandler);

//这里开始的代码会被立即执行下去，因为上面的语句不会被阻塞。

resultHandler.waitFor(5000);//最多阻塞等待5秒，超时返回。
```

可以使用waitFor来阻塞处理逻辑，比如上面的代码，在执行到waitFor时，会等待5秒再继续执行。 
之后，可以通过resultHandler.hasResult()、resultHandler.getExitValue()、resultHandler.getException()获得需要信息。 

注意：getException();得到的是Exec自己的异常，不是应用程序(比如JAVA)代码里面抛出的异常。

### 终止进程
通过Watchdog，可以终止正在运行的进程。
```java
final CommandLine cmdLine = CommandLine.parse("ping www.baidu.com -t");
final ExecuteWatchdog watchdog = new ExecuteWatchdog(Integer.MAX_VALUE);
final DefaultExecuteResultHandler resultHandler = new DefaultExecuteResultHandler();
DefaultExecutor executor = new DefaultExecutor();

executor.setWatchdog(watchdog);
executor.execute(cmdLine, resultHandler);

Thread.sleep(10000);//等进程执行一会，再终止它
System.out.println("--> Watchdog is watching ? " + watchdog.isWatching());
watchdog.destroyProcess();//终止进程
System.out.println("--> destroyProcess done.");
System.out.println("--> Watchdog is watching ? " + watchdog.isWatching());
System.out.println("--> Watchdog should have killed the process : " + watchdog.killedProcess());
System.out.println("--> wait result is : " + resultHandler.hasResult());
System.out.println("--> exit value is : " + resultHandler.getExitValue());
System.out.println("--> exception is : " + resultHandler.getException());

resultHandler.waitFor(5000);//等待5秒。下面加上上面的几个System.out，看看进程状态是什么。
```

### 获得进程的输出信息
可以在程序中，通过PumpStreamHandler，截获进程的各种输出，包括output 和 error stream。
```java
String cmdStr = "ping www.baidu.com";
final CommandLine cmdLine = CommandLine.parse(cmdStr);

DefaultExecutor executor = new DefaultExecutor();
ByteArrayOutputStream baos = new ByteArrayOutputStream();
executor.setStreamHandler(new PumpStreamHandler(baos, baos));

executor.setExitValue(1);
int exitValue = executor.execute(cmdLine);

//不同操作系统注意编码，否则结果乱码
final String result = baos.toString("gbk").trim();
System.out.println(result);//这个result就是ping输出的结果。如果是JAVA程序，抛出了异常，也被它获取。
```

### 网上搜的案例

#### 案例1
在程序中执行本来通过run.bat脚本执行的内容。
在run.bat中的内容：
```java
spark-submit   --class "SimpleApp"   --master local[4]   D:/study/sparkstudy/simple-project/target/simple-project-1.0.jar
```

通过程序完成run.bat脚本的内容

第一种：
```java
String line = "spark-submit.cmd   --class \"SimpleApp\"   --master local[4]  D:\\study\\sparkstudy\\simple-project\\target\\simple-project-1.0.jar";

 CommandLine cmdLine =   CommandLine.parse(line);
 DefaultExecutor executor = new DefaultExecutor();
 executor.setExitValues(null);
 ExecuteWatchdog watchdog = new ExecuteWatchdog(60000);
 executor.setWatchdog(watchdog);

 ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
 ByteArrayOutputStream errorStream = new ByteArrayOutputStream();
 PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream,errorStream);

 executor.setStreamHandler(streamHandler);
 executor.execute(cmdLine);
 String out = outputStream.toString("gbk");//获取程序外部程序执行结果
 String error = errorStream.toString("gbk");
```

第二种：
```java
File file = new File("D:\\study\\sparkstudy\\simple-project\\target\\simple-project-1.0.jar");
Map map = new HashMap();
map.put("FILE", file);
CommandLine cmdLine =  new CommandLine("spark-submit.cmd");
cmdLine.addArgument("--class");
cmdLine.addArgument("\"SimpleApp\"");
cmdLine.addArgument("--master");
cmdLine.addArgument("local[4]");
cmdLine.addArgument("${FILE}");
cmdLine.setSubstitutionMap(map);

DefaultExecutor executor = new DefaultExecutor();
executor.setExitValues(null);
ExecuteWatchdog watchdog = new ExecuteWatchdog(60000);
executor.setWatchdog(watchdog);

ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
ByteArrayOutputStream errorStream = new ByteArrayOutputStream();
PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream,errorStream);
executor.setStreamHandler(streamHandler);
executor.execute(cmdLine);
String out = outputStream.toString("gbk");//获取程序外部程序执行结果
String error = errorStream.toString("gbk");
```

#### 案例2
工具类ScriptUtil
```java
import org.apache.commons.exec.CommandLine;
import org.apache.commons.exec.DefaultExecutor;
import org.apache.commons.exec.PumpStreamHandler;
import org.springframework.util.StringUtils;

import java.io.ByteArrayOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * 1、内嵌编译器如"PythonInterpreter"无法引用扩展包，因此推荐使用java调用控制台进程方式"Runtime.getRuntime().execCmd()"来运行脚本(shell或python)；
 * 2、因为通过java调用控制台进程方式实现，需要保证目标机器PATH路径正确配置对应编译器；
 * 3、暂时脚本执行日志只能在脚本执行结束后一次性获取，无法保证实时性；因此为确保日志实时性，可改为将脚本打印的日志存储在指定的日志文件上；
 * 4、python 异常输出优先级高于标准输出，体现在Log文件中，因此推荐通过logging方式打日志保持和异常信息一致；否则用prinf日志顺序会错乱
 * <p>
 *
 * @author lism
 * @date 2018年11月8日10:02:30
 */
public class ScriptUtil {

    /**
     * make script file
     *
     * @param scriptFileName
     * @param content
     * @throws IOException
     */
    public static void markScriptFile(String scriptFileName, String content) throws Exception {
        // make file,   filePath/gluesource/666-123456789.py
        FileOutputStream fileOutputStream = null;
        try {
            fileOutputStream = new FileOutputStream(scriptFileName);
            fileOutputStream.write(content.getBytes("UTF-8"));
            fileOutputStream.close();
        } catch (Exception e) {
            throw e;
        } finally {
            if (fileOutputStream != null) {
                fileOutputStream.close();
            }
        }
    }

    /**
     * 日志文件输出方式
     * <p>
     * 优点：支持将目标数据实时输出到指定日志文件中去
     * 缺点：
     * 标准输出和错误输出优先级固定，可能和脚本中顺序不一致
     * Java无法实时获取
     *
     * @param command
     * @param scriptFile
     * @param logFile
     * @param params
     * @return
     * @throws IOException
     */
    public static int execToFile(String command, String scriptFile, String logFile, String... params) throws IOException {
        // 标准输出：print （null if watchdog timeout）
        // 错误输出：logging + 异常 （still exists if watchdog timeout）
        // 标准输入
        FileOutputStream fileOutputStream = null;
        try {
            fileOutputStream = new FileOutputStream(logFile, true);
            PumpStreamHandler streamHandler = new PumpStreamHandler(fileOutputStream, fileOutputStream, null);
            int exitValue = execCmd(command, scriptFile, params, streamHandler);
            return exitValue;
        } catch (Exception e) {
            return -1;
        } finally {
            if (fileOutputStream != null) {
                try {
                    fileOutputStream.close();
                } catch (IOException e) {
                }
            }
        }
    }

    public static int execCmd(String command, String scriptFile, String... params) throws IOException {
        PumpStreamHandler streamHandler = new PumpStreamHandler(new CollectingLogOutputStream());
        // command
        return execCmd(command, scriptFile, params, streamHandler);
    }

    public static int execCmd(String command, String scriptFile, String[] params, PumpStreamHandler streamHandler) throws IOException {
        CommandLine commandline = new CommandLine(command);
        if(!StringUtils.isEmpty(scriptFile)){
            commandline.addArgument(scriptFile);
        }
        if (params != null && params.length > 0) {
            commandline.addArguments(params);
        }
        // execCmd
        DefaultExecutor exec = new DefaultExecutor();
        exec.setExitValues(null);
        exec.setStreamHandler(streamHandler);
        int exitValue = exec.execute(commandline);// exit code: 0=success, 1=error
        return exitValue;
    }

    public static String execToString(String command, String scriptFile, String logFile, String... params) throws IOException {
        // 标准输出：print （null if watchdog timeout）
        // 错误输出：logging + 异常 （still exists if watchdog timeout）
        // 标准输入

        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();

        try {
            PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream, outputStream);
            execCmd(command, scriptFile, params, streamHandler);
        } catch (Exception e) {
            return null;
        } finally {
            if (outputStream != null) {
                try {
                    outputStream.close();
                } catch (IOException e) {
                }

            }
        }
        return outputStream.toString("gbk");
    }

}
```

如果不是要把脚本的日志输出到文件或者流中，而是实时处理每一行输出，需要实现LogOutputStream，类似下面的实现类CollectingLogOutputStream
```java
import org.apache.commons.exec.LogOutputStream;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.LinkedList;
import java.util.List;

/**
 * @author lism
 */
public class CollectingLogOutputStream extends LogOutputStream {

    private static Logger logger = LoggerFactory.getLogger(CollectingLogOutputStream.class);
    private final List<String> lines = new LinkedList<String>();
    @Override protected void processLine(String line, int level) {

        lines.add(line);
        logger.info("日志级别{}：{}",level,line);
    }   
    public List<String> getLines() {
        return lines;
    }
}
```

使用方式是将CollectingLogOutputStream实现类传给PumpStreamHandler当参数，这样就能实时处理脚本输出了
```java
public static int execCmd(String command, String scriptFile, String... params) throws IOException {
    PumpStreamHandler streamHandler = new PumpStreamHandler(new CollectingLogOutputStream());
    // command
    return execCmd(command, scriptFile, params, streamHandler);
}
```

#### 案例3
```java
package io.selendroid.standalone.io;

import io.selendroid.standalone.exceptions.DeviceOfflineException;
import io.selendroid.standalone.exceptions.ShellCommandException;

import java.util.Map;
import java.util.logging.Level;
import java.util.logging.Logger;

import org.apache.commons.exec.CommandLine;
import org.apache.commons.exec.DefaultExecuteResultHandler;
import org.apache.commons.exec.DefaultExecutor;
import org.apache.commons.exec.ExecuteResultHandler;
import org.apache.commons.exec.ExecuteWatchdog;
import org.apache.commons.exec.LogOutputStream;
import org.apache.commons.exec.PumpStreamHandler;
import org.apache.commons.exec.environment.EnvironmentUtils;

public class ShellCommand {

  private static final Logger log = Logger.getLogger(ShellCommand.class.getName());

  public static String exec(CommandLine commandLine) throws ShellCommandException {
    return exec(commandLine, 20000);
  }

  public static String exec(CommandLine commandline, long timeoutInMillies)
      throws ShellCommandException {
    log.info("Executing shell command: " + commandline);
    PrintingLogOutputStream outputStream = new PrintingLogOutputStream();
    DefaultExecutor exec = new DefaultExecutor();
    exec.setWatchdog(new ExecuteWatchdog(timeoutInMillies));
    PumpStreamHandler streamHandler = new PumpStreamHandler(outputStream);
    exec.setStreamHandler(streamHandler);
    try {
      exec.execute(commandline);
    } catch (Exception e) {
      log.log(Level.SEVERE, "Error executing command: " + commandline, e);
      if (e.getMessage().contains("device offline")) {
        throw new DeviceOfflineException(e);
      }
      throw new ShellCommandException(
          "Error executing shell command: " + commandline, new ShellCommandException(outputStream.getOutput()));
    }
    String result = outputStream.getOutput().trim();
    log.info("Shell command output\n-->\n" + result + "\n<--");
    return result;
  }

  public static void execAsync(CommandLine commandline) throws ShellCommandException {
    execAsync(null, commandline);
  }

  public static void execAsync(String display, CommandLine commandline)
      throws ShellCommandException {
    log.info("executing async command: " + commandline);
    DefaultExecutor exec = new DefaultExecutor();

    ExecuteResultHandler handler = new DefaultExecuteResultHandler();
    PumpStreamHandler streamHandler = new PumpStreamHandler(new PrintingLogOutputStream());
    exec.setStreamHandler(streamHandler);
    try {
      if (display == null || display.isEmpty()) {
        exec.execute(commandline, handler);
      } else {
        Map env = EnvironmentUtils.getProcEnvironment();
        EnvironmentUtils.addVariableToEnvironment(env, "DISPLAY=:" + display);

        exec.execute(commandline, env, handler);
      }
    } catch (Exception e) {
      log.log(Level.SEVERE, "Error executing command: " + commandline, e);
      throw new ShellCommandException("Error executing shell command: " + commandline, e);
    }
  }

  private static class PrintingLogOutputStream extends LogOutputStream {

    private StringBuilder output = new StringBuilder();

    @Override
    protected void processLine(String line, int level) {
      log.fine("OUTPUT FROM PROCESS: " + line);

      output.append(line).append("\n");
    }

    public String getOutput() {
      return output.toString();
    }
  }
}
```

