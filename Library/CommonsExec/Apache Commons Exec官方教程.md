## Apache Commons Exec官方教程
如果你看到这篇文章，相信你肯定使用java新建过子进程，去执行shell命令，并且肯定耗费了不少时间。你可能会认为java自身的Runtime.exec()很简单，而Apache Commons Exec太过臃肿，纯粹是在浪费时间。

但是，我在使用Runtime.exec()的过程时，经历了一系列痛苦的过程。一起来看下commons exec是怎么把这一过程变简单的。

### 第一个例子
下面是一个简单的例子，我们调用了命令行 AcorRd32.exe，当然前提是你的系统中有这个东西。
```java
String line = "AcroRd32.exe /p /h " + file.getAbsolutePath();
CommandLine cmdLine = CommandLine.parse(line);
DefaultExecutor executor = new DefaultExecutor();
int exitValue = executor.execute(cmdLine);
```

上面的代码执行后，我们打印出了pdf文档中的内容，但是运行抛出异常了。原因是Acrobat Reader运行结束后，它的返回值是1，而一般情况下认为返回值为1是错误的，所以我们需要修改一下代码。
```java
String line = "AcroRd32.exe /p /h " + file.getAbsolutePath();
CommandLine cmdLine = CommandLine.parse(line);
DefaultExecutor executor = new DefaultExecutor();
executor.setExitValue(1);
int exitValue = executor.execute(cmdLine);
```

### 是否使用Watchdog
我们的程序输出了pdf内容。但是如果在输出过程中，我们的程序因为各种原因hang住了，怎么办？启动任务很简单，但是如果子进程失败了怎么办。Commons-exec提供了watchdog来解决这一问题。下面的代码中，如果子进程运行超过6秒，那么就强制结束它。
```java
String line = "AcroRd32.exe /p /h " + file.getAbsolutePath();
CommandLine cmdLine = CommandLine.parse(line);
DefaultExecutor executor = new DefaultExecutor();
executor.setExitValue(1);
ExecuteWatchdog watchdog = new ExecuteWatchdog(60000);
executor.setWatchdog(watchdog);
int exitValue = executor.execute(cmdLine);
```

### 引号很有用
上面的代码还有个问题，如果输入的文件路径中含有空格，会有异常。
```java
    > AcroRd32.exe /p /h C:\Document And Settings\documents\432432.pdf  
```

解决方法是，使用引号将文件路径括起来：
```java
String line = "AcroRd32.exe /p /h \"" + file.getAbsolutePath() + "\"";
CommandLine cmdLine = CommandLine.parse(line);
DefaultExecutor executor = new DefaultExecutor();
executor.setExitValue(1);
ExecuteWatchdog watchdog = new ExecuteWatchdog(60000);
executor.setWatchdog(watchdog);
int exitValue = executor.execute(cmdLine);
```

### 逐步生成CommandLine对象
上面的引号问题，是因为commons-exec错误地分割了我们的路径字符串，导致文件路径错误，我们可是使用CommandLine对象提供的方法，来避免手动拼装命令。
```java
Map map = new HashMap();
map.put("file", new File("invoice.pdf"));
CommandLine cmdLine = new CommandLine("AcroRd32.exe");
cmdLine.addArgument("/p");
cmdLine.addArgument("/h");
cmdLine.addArgument("${file}");
cmdLine.setSubstitutionMap(map);
DefaultExecutor executor = new DefaultExecutor();
executor.setExitValue(1);
ExecuteWatchdog watchdog = new ExecuteWatchdog(60000);
executor.setWatchdog(watchdog);
int exitValue = executor.execute(cmdLine);
```

### 异步执行
现在为止，我们的程序可以良好运行了，但是还不适用生产，原因是它是阻塞的，在子进程结束前，它会一直阻塞我们的主进程。现在我们要把它变成非阻塞的。下面的代码，我们创建了一个ExecuteResultHandler对象，将它传给Executor，让它可以异步执行任务。**ResultHandler会接受子进程抛出的一切异常。**
```java
CommandLine cmdLine = new CommandLine("AcroRd32.exe");
cmdLine.addArgument("/p");
cmdLine.addArgument("/h");
cmdLine.addArgument("${file}");
HashMap map = new HashMap();
map.put("file", new File("invoice.pdf"));
commandLine.setSubstitutionMap(map);

DefaultExecuteResultHandler resultHandler = new         DefaultExecuteResultHandler();

ExecuteWatchdog watchdog = new ExecuteWatchdog(60*1000);
Executor executor = new DefaultExecutor();
executor.setExitValue(1);
executor.setWatchdog(watchdog);
executor.execute(cmdLine, resultHandler);

//一段时间后，resultHandler的回调方法会被唤醒，我们可以查看其返回值
int exitValue = resultHandler.waitFor();
```

### 完整的代码
```java
package org.apache.commons.exec;

import static org.junit.Assert.fail;

import java.io.File;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

import org.junit.Test;

/**
 * An example based on the tutorial where the user can can safely play with
 * <ul>
 *  <li>blocking or non-blocking print jobs
 *  <li>with print job timeouts to trigger the {@code ExecuteWatchdog}
 *  <li>with the {@code exitValue} returned from the print script
 * </ul>
 *
 */
public class TutorialTest {

    /** the directory to pick up the test scripts */
    private final File testDir = new File("src/test/scripts");

    /** simulates a PDF print job */
    private final File acroRd32Script = TestUtil.resolveScriptForOS(testDir + "/acrord32");

    @Test
    public void testTutorialExample() throws Exception {

        final long printJobTimeout = 15000;
        final boolean printInBackground = false;
        final File pdfFile = new File("/Documents and Settings/foo.pdf");

        PrintResultHandler printResult;

        try {
            // printing takes around 10 seconds            
            System.out.println("[main] Preparing print job ...");
            printResult = print(pdfFile, printJobTimeout, printInBackground);
            System.out.println("[main] Successfully sent the print job ...");
        }
        catch (final Exception e) {
            e.printStackTrace();
            fail("[main] Printing of the following document failed : " + pdfFile.getAbsolutePath());
            throw e;
        }

        // come back to check the print result
        System.out.println("[main] Test is exiting but waiting for the print job to finish...");
        printResult.waitFor();
        System.out.println("[main] The print job has finished ...");
    }

    /**
     * Simulate printing a PDF document.
     *
     * @param file the file to print
     * @param printJobTimeout the printJobTimeout (ms) before the watchdog terminates the print process
     * @param printInBackground printing done in the background or blocking
     * @return a print result handler (implementing a future)
     * @throws IOException the test failed
     */
    public PrintResultHandler print(final File file, final long printJobTimeout, final boolean printInBackground)
            throws IOException {

        int exitValue;
        ExecuteWatchdog watchdog = null;
        PrintResultHandler resultHandler;

        // build up the command line to using a 'java.io.File'
        final Map<String, File> map = new HashMap<String, File>();
        map.put("file", file);
        final CommandLine commandLine = new CommandLine(acroRd32Script);
        commandLine.addArgument("/p");
        commandLine.addArgument("/h");
        commandLine.addArgument("${file}");
        commandLine.setSubstitutionMap(map);

        // create the executor and consider the exitValue '1' as success
        final Executor executor = new DefaultExecutor();
        executor.setExitValue(1);
        
        // create a watchdog if requested
        if (printJobTimeout > 0) {
            watchdog = new ExecuteWatchdog(printJobTimeout);
            executor.setWatchdog(watchdog);
        }

        // pass a "ExecuteResultHandler" when doing background printing
        if (printInBackground) {
            System.out.println("[print] Executing non-blocking print job  ...");
            resultHandler = new PrintResultHandler(watchdog);
            executor.execute(commandLine, resultHandler);
        }
        else {
            System.out.println("[print] Executing blocking print job  ...");
            exitValue = executor.execute(commandLine);
            resultHandler = new PrintResultHandler(exitValue);
        }

        return resultHandler;
    }

    private class PrintResultHandler extends DefaultExecuteResultHandler {

        private ExecuteWatchdog watchdog;

        public PrintResultHandler(final ExecuteWatchdog watchdog)
        {
            this.watchdog = watchdog;
        }

        public PrintResultHandler(final int exitValue) {
            super.onProcessComplete(exitValue);
        }
        
        @Override
        public void onProcessComplete(final int exitValue) {
            super.onProcessComplete(exitValue);
            System.out.println("[resultHandler] The document was successfully printed ...");
        }

        @Override
        public void onProcessFailed(final ExecuteException e) {
            super.onProcessFailed(e);
            if (watchdog != null && watchdog.killedProcess()) {
                System.err.println("[resultHandler] The print process timed out");
            }
            else {
                System.err.println("[resultHandler] The print process failed to do : " + e.getMessage());
            }
        }
    }
}
```











```java

```

