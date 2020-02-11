## Commons Exec源码阅读
Commons Exec的代码不多，所以顺便看看Commons Exec的代码，记录一下。

### DefaultExecutor类
DefaultExecutor类为最主要的类，The default class to start a subprocess. 

最基本的使用：
```java
Executor exec = new DefaultExecutor();
CommandLine cl = new CommandLine("ls -l");
int exitvalue = exec.execute(cl);
```

#### DefaultExecutor的属性和构造函数

```java
//Executor是一个接口，定义了运行外部的主要抽象方法
public class DefaultExecutor implements Executor {

    /** taking care of output and error stream */
    private ExecuteStreamHandler streamHandler;//用于处理子进程的输入输出流，默认会用System.out, System.err消费子进程的输出流

    /** the working directory of the process */
    private File workingDirectory;//用于设置工作目录，默认为当前目录

    /** monitoring of long running processes */
    private ExecuteWatchdog watchdog;//看门狗，用于设置超时，默认为null

    /** the exit values considered to be successful */
    private int[] exitValues;//被认为为正常退出的exitValue，默认0为成功退出

    /** launches the command in a new process */
    private final CommandLauncher launcher;//在子进程中调用命令，其实就是调用Runtime.getRuntime().exe()

    /** optional cleanup of started processes */ 
    private ProcessDestroyer processDestroyer;//默认为null，

    /** worker thread for asynchronous execution */
    private Thread executorThread;//异步执行命令的线程

    /** the first exception being caught to be thrown to the caller */
    private IOException exceptionCaught;//调用命令捕获到的第一个异常

    public DefaultExecutor() {
        //默认的流处理器，复制子进程的标准输出和标准错误到System.out, System.err
        this.streamHandler = new PumpStreamHandler();
        //启动子进程中调用命令，其实就是调用Runtime.getRuntime().exe()
        this.launcher = CommandLauncherFactory.createVMLauncher();
        //默认0为成功退出
        this.exitValues = new int[0];
        //工作目录默认为当前目录
        this.workingDirectory = new File(".");
        this.exceptionCaught = null;
    }
    
}
```

#### 同步执行命令和异步执行命令
可以看到无论是同步还是异步的执行命令，最终执行命令的方法为`executeInternal(command, environment, workingDirectory, streamHandler)`

异步方式和同步方式的不同就是新启了一个线程执行命令，并在进程完成或者抛出异常时调用ExecuteResultHandler的onProcessComplete、onProcessFailed回调函数。

ExecuteResultHandler可以监控子进程的完成状态，获得exitValue。
```java
//同步执行命令
public int execute(final CommandLine command) throws ExecuteException,
        IOException {
    return execute(command, (Map<String, String>) null);
}

public int execute(final CommandLine command, final Map<String, String> environment)
        throws ExecuteException, IOException {

    if (workingDirectory != null && !workingDirectory.exists()) {
        throw new IOException(workingDirectory + " doesn't exist.");
    }
    
    return executeInternal(command, environment, workingDirectory, streamHandler);

}

//异步执行命令
public void execute(final CommandLine command, final ExecuteResultHandler handler)
        throws ExecuteException, IOException {
    execute(command, null, handler);
}

public void execute(final CommandLine command, final Map<String, String> environment,
        final ExecuteResultHandler handler) throws ExecuteException, IOException {

    if (workingDirectory != null && !workingDirectory.exists()) {
        throw new IOException(workingDirectory + " doesn't exist.");
    }

    if (watchdog != null) {
        watchdog.setProcessNotStarted();
    }

    final Runnable runnable = new Runnable()
    {
        public void run()
        {
            int exitValue = Executor.INVALID_EXITVALUE;
            try {
                exitValue = executeInternal(command, environment, workingDirectory, streamHandler);
                handler.onProcessComplete(exitValue);
            } catch (final ExecuteException e) {
                handler.onProcessFailed(e);
            } catch (final Exception e) {
                handler.onProcessFailed(new ExecuteException("Execution failed", exitValue, e));
            }
        }
    };

    this.executorThread = createThread(runnable, "Exec Default Executor");
    getExecutorThread().start();
}
```
#### executeInternal方法
```java
/**
 * Execute an internal process. If the executing thread is interrupted while waiting for the
 * child process to return the child process will be killed.
 * 阻塞等待子进程结束，如果此线程在wait时被中断，则会杀死子进程
 *
 * @param command the command to execute
 * @param environment the execution environment
 * @param dir the working directory
 * @param streams process the streams (in, out, err) of the process
 * @return the exit code of the process
 * @throws IOException executing the process failed
 */
private int executeInternal(final CommandLine command, final Map<String, String> environment,
        final File dir, final ExecuteStreamHandler streams) throws IOException {

    //没用处
    setExceptionCaught(null);

    //Runtime.getRuntime().exe()
    final Process process = this.launch(command, environment, dir);

    try {
        //映射子进程的输入流，思考此处为啥是process.getOutputStream()?应为这是相对于我们的应用程序来说的子进程的输入是输出流，也只有在输入输出流之间才能拷贝流
        streams.setProcessInputStream(process.getOutputStream());
        //映射子进程的标准输出流
        streams.setProcessOutputStream(process.getInputStream());
        //映射子进程的标准错误输出流
        streams.setProcessErrorStream(process.getErrorStream());
    } catch (final IOException e) {
        process.destroy();
        throw e;
    }

    //会分别启动一个线程消费子进程的输入流、标准输出流标准错误输出流。如果流为null，则不会启动线程。默认只消费标准输出、标准错误输出。
    streams.start();

    try {

        // add the process to the list of those to destroy if the VM exits
        // 暂时没用到
        if (this.getProcessDestroyer() != null) {
          this.getProcessDestroyer().add(process);
        }

        // associate the watchdog with the newly created process
        // 看门狗开始工作，会启动一个线程监控超时。和子进程关联起来，好停止子进程
        if (watchdog != null) {
            watchdog.start(process);
        }

        int exitValue = Executor.INVALID_EXITVALUE;

        try {
            //阻塞等待子进程结束
            exitValue = process.waitFor();
        } catch (final InterruptedException e) {
            //如果此线程在wait时被中断，则会杀死子进程
            process.destroy();
        }
        finally {
            // see http://bugs.sun.com/view_bug.do?bug_id=6420270
            // see https://issues.apache.org/jira/browse/EXEC-46
            // Process.waitFor should clear interrupt status when throwing InterruptedException
            // but we have to do that manually
            Thread.interrupted();
        }            

        if (watchdog != null) {
            //关闭看门狗
            watchdog.stop();
        }

        try {
            //关闭ExecuteStreamHandler的流处理线程
            streams.stop();
        }
        catch (final IOException e) {
            setExceptionCaught(e);
        }

        // 关闭子进程的各个流
        closeProcessStreams(process);

        if (getExceptionCaught() != null) {
            throw getExceptionCaught();
        }

        if (watchdog != null) {
            try {
                watchdog.checkException();
            } catch (final IOException e) {
                throw e;
            } catch (final Exception e) {
                throw new IOException(e.getMessage());
            }
        }

        if (this.isFailure(exitValue)) {
            throw new ExecuteException("Process exited with an error: " + exitValue, exitValue);
        }

        return exitValue;
    } finally {
        // remove the process to the list of those to destroy if the VM exits
        if (this.getProcessDestroyer() != null) {
          this.getProcessDestroyer().remove(process);
        }
    }
}
```

### PumpStreamHandler类
PumpStreamHandler类用于消费子进程的输入输出流，默认会用System.out, System.err消费子进程的输出流

会分别启动一个线程消费子进程的输入流、标准输出流标准错误输出流。如果流为null，则不会启动线程。默认只消费标准输出、标准错误输出。

自定义子进程标准输出和标准错误的输出：
```java
DefaultExecutor executor = new DefaultExecutor();
ByteArrayOutputStream baos = new ByteArrayOutputStream();
executor.setStreamHandler(new PumpStreamHandler(baos, baos));
```

#### PumpStreamHandler的属性和构造函数
```java
/**
 * Copies standard output and error of sub-processes to standard output and error
 * of the parent process. If output or error stream are set to null, any feedback
 * from that stream will be lost.
 *
 * @version $Id: PumpStreamHandler.java 1557263 2014-01-10 21:18:09Z ggregory $
 */
public class PumpStreamHandler implements ExecuteStreamHandler {

    private static final long STOP_TIMEOUT_ADDITION = 2000L;//停止线程超时时间的额外增加

    private Thread outputThread;//消费子进程标准输出线程

    private Thread errorThread;//消费子进程标准错误线程

    private Thread inputThread;//消费子进程标准输入线程

    private final OutputStream out;//用于消费子进程标准输出

    private final OutputStream err;//用于消费子进程标准错误

    private final InputStream input;//用于消费子进程标准输入

    private InputStreamPumper inputStreamPumper;//消费子进程标准输入线程，专门处理input = System.in的情况

    /** the timeout in ms the implementation waits when stopping the pumper threads */
    private long stopTimeout;//停止消费线程的超时时间，默认为一直阻塞等待

    /** the last exception being caught */
    private IOException caught = null;//捕获到的第一个异常

    public PumpStreamHandler() {
        //默认的构造函数
        this(System.out, System.err);
    }

    public PumpStreamHandler(final OutputStream outAndErr) {
        //标准输出和标准错误映射到一个流
        this(outAndErr, outAndErr);
    }

    public PumpStreamHandler(final OutputStream out, final OutputStream err) {       
        this(out, err, null);
    }

    public PumpStreamHandler(final OutputStream out, final OutputStream err, final InputStream input) {
        this.out = out;
        this.err = err;
        this.input = input;
    }
    
}
```

#### PumpStreamHandler的方法
executeInternal中有关PumpStreamHandler的调用
```java
private int executeInternal(final CommandLine command, final Map<String, String> environment,
        final File dir, final ExecuteStreamHandler streams) throws IOException {
    final Process process = this.launch(command, environment, dir);

    try {
        //映射各个流，消费流线程的创建
        streams.setProcessInputStream(process.getOutputStream());
        streams.setProcessOutputStream(process.getInputStream());
        streams.setProcessErrorStream(process.getErrorStream());
    } catch (final IOException e) {
        process.destroy();
        throw e;
    }

    streams.start();//消费流线程的启动

    try {
        ...
        try {
            exitValue = process.waitFor();
        } catch (final InterruptedException e) {
            process.destroy();
        }
        finally {
            Thread.interrupted();
        }            

        ...

        try {
            streams.stop();//
        }
        catch (final IOException e) {
            setExceptionCaught(e);
        }

        return exitValue;
    } finally {
        ```
    }
}
```

映射流以及消费流线程的创建的方法：
```java
public void setProcessOutputStream(final InputStream is) {
    if (out != null) {
        createProcessOutputPump(is, out);
    }
}

public void setProcessErrorStream(final InputStream is) {
    if (err != null) {
        createProcessErrorPump(is, err);
    }
}


public void setProcessInputStream(final OutputStream os) {
    if (input != null) {
        if (input == System.in) {
            inputThread = createSystemInPump(input, os);//单独处理
        } else {
            inputThread = createPump(input, os, true);
        }
    } 
}

protected void createProcessOutputPump(final InputStream is, final OutputStream os) {
    outputThread = createPump(is, os);
}

protected void createProcessErrorPump(final InputStream is, final OutputStream os) {
    errorThread = createPump(is, os);
}

protected Thread createPump(final InputStream is, final OutputStream os) {
    final boolean closeWhenExhausted = os instanceof PipedOutputStream ? true : false;
    return createPump(is, os, closeWhenExhausted);
}

protected Thread createPump(final InputStream is, final OutputStream os, final boolean closeWhenExhausted) {
     final Thread result = new Thread(new StreamPumper(is, os, closeWhenExhausted), "Exec Stream Pumper");
     result.setDaemon(true);//设置为守护线程
     return result;
 }
```

消费流线程的启动：
```java
public void start() {
    if (outputThread != null) {
        outputThread.start();
    }
    if (errorThread != null) {
        errorThread.start();
    }
    if (inputThread != null) {
        inputThread.start();
    }
}
```

消费流线程的停止：
```java
/**
 * Stop pumping the streams. When a timeout is specified it it is not guaranteed that the
 * pumper threads are cleanly terminated.
 */
public void stop() throws IOException {

    if (inputStreamPumper != null) {
        inputStreamPumper.stopProcessing();
    }

    //停止各个线程：默认阻塞等待线程结束，设置了超时就不能保证线程一定结束。
    stopThread(outputThread, stopTimeout);
    stopThread(errorThread, stopTimeout);
    stopThread(inputThread, stopTimeout);

    if (err != null && err != out) {
        try {
            err.flush();
        } catch (final IOException e) {
            final String msg = "Got exception while flushing the error stream : " + e.getMessage();
            DebugUtils.handleException(msg, e);
        }
    }

    if (out != null) {
        try {
            out.flush();
        } catch (final IOException e) {
            final String msg = "Got exception while flushing the output stream";
            DebugUtils.handleException(msg, e);
        }
    }

    if (caught != null) {
        throw caught;
    }
}

/**
 * Stopping a pumper thread. The implementation actually waits
 * longer than specified in 'timeout' to detect if the timeout
 * was indeed exceeded. If the timeout was exceeded an IOException
 * is created to be thrown to the caller.
 *
 * @param thread  the thread to be stopped
 * @param timeout the time in ms to wait to join
 */
protected void stopThread(final Thread thread, final long timeout) {

    if (thread != null) {
        try {
            //默认阻塞等待线程结束，设置了超时就不能保证线程一定结束。
            if (timeout == 0) {
                thread.join();
            } else {
                final long timeToWait = timeout + STOP_TIMEOUT_ADDITION;
                final long startTime = System.currentTimeMillis();
                thread.join(timeToWait);
                if (!(System.currentTimeMillis() < startTime + timeToWait)) {
                    final String msg = "The stop timeout of " + timeout + " ms was exceeded";
                    caught = new ExecuteException(msg, Executor.INVALID_EXITVALUE);
                }
            }
        } catch (final InterruptedException e) {
            thread.interrupt();
        }
    }
}
```

#### StreamPumper
消费流的线程
```java
/**
 * Copies all data from an input stream to an output stream.
 *
 * @version $Id: StreamPumper.java 1557263 2014-01-10 21:18:09Z ggregory $
 */
public class StreamPumper implements Runnable {

    /** the default size of the internal buffer for copying the streams */
    private static final int DEFAULT_SIZE = 1024;

    /** the input stream to pump from */
    private final InputStream is;

    /** the output stream to pmp into */
    private final OutputStream os;

    /** the size of the internal buffer for copying the streams */ 
    private final int size;//读取流buffer的size

    /** was the end of the stream reached */
    private boolean finished;

    /** close the output stream when exhausted */
    private final boolean closeWhenExhausted;//最后是否关闭输出流
    
    /**
     * Create a new stream pumper.
     * 
     * @param is input stream to read data from
     * @param os output stream to write data to.
     * @param closeWhenExhausted if true, the output stream will be closed when the input is exhausted.
     */
    public StreamPumper(final InputStream is, final OutputStream os,
            final boolean closeWhenExhausted) {
        this.is = is;
        this.os = os;
        this.size = DEFAULT_SIZE;
        this.closeWhenExhausted = closeWhenExhausted;
    }

    /**
     * Create a new stream pumper.
     *
     * @param is input stream to read data from
     * @param os output stream to write data to.
     * @param closeWhenExhausted if true, the output stream will be closed when the input is exhausted.
     * @param size the size of the internal buffer for copying the streams
     */
    public StreamPumper(final InputStream is, final OutputStream os,
            final boolean closeWhenExhausted, final int size) {
        this.is = is;
        this.os = os;
        this.size = size > 0 ? size : DEFAULT_SIZE;
        this.closeWhenExhausted = closeWhenExhausted;
    }

    /**
     * Create a new stream pumper.
     * 
     * @param is input stream to read data from
     * @param os output stream to write data to.
     */
    public StreamPumper(final InputStream is, final OutputStream os) {
        this(is, os, false);
    }

    /**
     * Copies data from the input stream to the output stream. Terminates as
     * soon as the input stream is closed or an error occurs.
     */
    public void run() {
        synchronized (this) {
            // Just in case this object is reused in the future
            finished = false;
        }

        final byte[] buf = new byte[this.size];

        int length;
        try {
            while ((length = is.read(buf)) > 0) {
                os.write(buf, 0, length);
            }
        } catch (final Exception e) {
            // nothing to do - happens quite often with watchdog
        } finally {
            if (closeWhenExhausted) {
                try {
                    os.close();
                } catch (final IOException e) {
                    final String msg = "Got exception while closing exhausted output stream";
                    DebugUtils.handleException(msg ,e);
                }
            }
            synchronized (this) {
                finished = true;
                notifyAll();
            }
        }
    }

    /**
     * Tells whether the end of the stream has been reached.
     * 
     * @return true is the stream has been exhausted.
     */
    public synchronized boolean isFinished() {
        return finished;
    }

    /**
     * This method blocks until the stream pumper finishes.
     * 
     * @exception InterruptedException
     *                if any thread interrupted the current thread before or while the current thread was waiting for a
     *                notification.
     * @see #isFinished()
     */
    public synchronized void waitFor() throws InterruptedException {
        while (!isFinished()) {
            wait();
        }
    }
}
```

### ExecuteWatchdog类和Watchdog类
设置超时，监控子进程超时结束子进程。Watchdog为线程，是实际监控超时的。

executeInternal中
```java
final Process process = this.launch(command, environment, dir);

if (watchdog != null) {
    watchdog.start(process);
}

exitValue = process.waitFor();

if (watchdog != null) {
    watchdog.stop();
}
```

#### ExecuteWatchdog
```java
public class ExecuteWatchdog implements TimeoutObserver {

    /** The marker for an infinite timeout */
    public static final long INFINITE_TIMEOUT = -1;
    
    /** The process to execute and watch for duration. */
    private Process process;

    /** Is a user-supplied timeout in use */
    private final boolean hasWatchdog;

    /** Say whether or not the watchdog is currently monitoring a process. */
    private boolean watch;

    /** Exception that might be thrown during the process execution. */
    private Exception caught;

    /** Say whether or not the process was killed due to running overtime. */
    private boolean killedProcess;

    /** Will tell us whether timeout has occurred. */
    private final Watchdog watchdog;

    /** Indicates that the process is verified as started */
    private volatile boolean processStarted;

    /**
     * Creates a new watchdog with a given timeout.
     * 
     * @param timeout
     *            the timeout for the process in milliseconds. It must be
     *            greater than 0 or 'INFINITE_TIMEOUT'
     */
    public ExecuteWatchdog(final long timeout) {
        this.killedProcess = false;
        this.watch = false;
        this.hasWatchdog = timeout != INFINITE_TIMEOUT;
        this.processStarted = false;
        if (this.hasWatchdog) {
            this.watchdog = new Watchdog(timeout);
            this.watchdog.addTimeoutObserver(this);
        }
        else {
            this.watchdog = null;
        }
    }

    /**
     * Watches the given process and terminates it, if it runs for too long. All
     * information from the previous run are reset.
     * 
     * @param processToMonitor
     *            the process to monitor. It cannot be {@code null}
     * @throws IllegalStateException
     *             if a process is still being monitored.
     */
    public synchronized void start(final Process processToMonitor) {
        if (processToMonitor == null) {
            throw new NullPointerException("process is null.");
        }
        if (this.process != null) {
            throw new IllegalStateException("Already running.");
        }
        this.caught = null;
        this.killedProcess = false;
        this.watch = true;
        this.process = processToMonitor;
        this.processStarted = true;
        this.notifyAll();
        if (this.hasWatchdog) {
            watchdog.start();
        }
    }

    /**
     * Stops the watcher. It will notify all threads possibly waiting on this
     * object.
     */
    public synchronized void stop() {
        if (hasWatchdog) {
            watchdog.stop();
        }
        watch = false;
        process = null;
    }

    /**
     * Destroys the running process manually.
     */
    public synchronized void destroyProcess() {
        ensureStarted();
        this.timeoutOccured(null);
        this.stop();
    }

    /**
     * Called after watchdog has finished.
     */
    public synchronized void timeoutOccured(final Watchdog w) {
        try {
            try {
                // We must check if the process was not stopped
                // before being here
                if (process != null) {
                    process.exitValue();
                }
            } catch (final IllegalThreadStateException itse) {
                // the process is not terminated, if this is really
                // a timeout and not a manual stop then destroy it.
                if (watch) {
                    killedProcess = true;
                    process.destroy();
                }
            }
        } catch (final Exception e) {
            caught = e;
            DebugUtils.handleException("Getting the exit value of the process failed", e);
        } finally {
            cleanUp();
        }
    }


    /**
     * This method will rethrow the exception that was possibly caught during
     * the run of the process. It will only remains valid once the process has
     * been terminated either by 'error', timeout or manual intervention.
     * Information will be discarded once a new process is ran.
     * 
     * @throws Exception
     *             a wrapped exception over the one that was silently swallowed
     *             and stored during the process run.
     */
    public synchronized void checkException() throws Exception {
        if (caught != null) {
            throw caught;
        }
    }

    /**
     * Indicates whether or not the watchdog is still monitoring the process.
     * 
     * @return {@code true} if the process is still running, otherwise
     *         {@code false}.
     */
    public synchronized boolean isWatching() {
        ensureStarted();
        return watch;
    }

    /**
     * Indicates whether the last process run was killed.
     * 
     * @return {@code true} if the process was killed
     *         {@code false}.
     */
    public synchronized boolean killedProcess() {
        return killedProcess;
    }

    /**
     * reset the monitor flag and the process.
     */
    protected synchronized void cleanUp() {
        watch = false;
        process = null;
    }

    void setProcessNotStarted() {
        processStarted = false;
    }

    /**
     * Ensures that the process is started, so we do not race with asynch execution.
     * The caller of this method must be holding the lock on this
     */
    private void ensureStarted() {
        while (!processStarted) {
            try {
                this.wait();
            } catch (final InterruptedException e) {
                throw new RuntimeException(e.getMessage());
            }
        }
    }
}
```

TimeoutObserver接口：
```java
/**
 * Interface for classes that want to be notified by Watchdog.
 * 
 * @see org.apache.commons.exec.Watchdog
 *
 * @version $Id: TimeoutObserver.java 1556869 2014-01-09 16:51:11Z britter $
 */
public interface TimeoutObserver {

    /**
     * Called when the watchdog times out.
     * 
     * @param w the watchdog that timed out.
     */
    void timeoutOccured(Watchdog w);
}
```

#### Watchdog
```java
public class Watchdog implements Runnable {

    private final Vector<TimeoutObserver> observers = new Vector<TimeoutObserver>(1);

    private final long timeout;

    private boolean stopped = false;

    public Watchdog(final long timeout) {
        if (timeout < 1) {
            throw new IllegalArgumentException("timeout must not be less than 1.");
        }
        this.timeout = timeout;
    }

    public void addTimeoutObserver(final TimeoutObserver to) {
        observers.addElement(to);
    }

    public void removeTimeoutObserver(final TimeoutObserver to) {
        observers.removeElement(to);
    }

    protected final void fireTimeoutOccured() {
        final Enumeration<TimeoutObserver> e = observers.elements();
        while (e.hasMoreElements()) {
            e.nextElement().timeoutOccured(this);
        }
    }

    public synchronized void start() {
        stopped = false;
        final Thread t = new Thread(this, "WATCHDOG");
        t.setDaemon(true);
        t.start();
    }

    public synchronized void stop() {
        stopped = true;
        notifyAll();
    }

    public void run() {
        final long startTime = System.currentTimeMillis();
        boolean isWaiting;
        synchronized (this) {
            long timeLeft = timeout - (System.currentTimeMillis() - startTime);
            isWaiting = timeLeft > 0;
            while (!stopped && isWaiting) {
                try {
                    wait(timeLeft);
                } catch (final InterruptedException e) {
                }
                timeLeft = timeout - (System.currentTimeMillis() - startTime);
                isWaiting = timeLeft > 0;
            }
        }

        // notify the listeners outside of the synchronized block (see EXEC-60)
        if (!isWaiting) {
            fireTimeoutOccured();
        }
    }
    
}
```

