## Java运行cmd命令直接导出.sql文件
`https://www.cnblogs.com/zhengbin/p/4967330.html`

Java中的Runtime.getRuntime().exec(commandStr)可以调用执行cmd命令
```java
import java.io.File;
import java.text.SimpleDateFormat;
import java.util.Date;


public class ExportSqlUtil {
    public static void main(String[] args) {
        try {
            backup("root","950906","station");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static boolean backup(String username,String password,String database){
        boolean bool = false;
        String sqlFilename = database+"_" + new SimpleDateFormat("yyyy-MM-dd_HH-mm-ss").format(new Date()) + ".sql";
        String cmd = "mysqldump -u "+username+" -p"+password+" --opt "+database+" > d:/"+sqlFilename;
        
        try {
            Process p = Runtime.getRuntime().exec("cmd /C" + cmd);
            p.waitFor();
            bool = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return bool;
    }
}
```
其中Process新建进程p，表示当前线程等待process这个线程执行完毕后，继续向下执行。

## Java执行CMD命令
`https://www.cnblogs.com/leodaxin/p/8991628.html`

```java
public static void runtimeCommand() throws Exception {
    Process process = Runtime.getRuntime().exec("cmd.exe /c dir");
    int status = process.waitFor();

    System.out.println(status);
    InputStream in = process.getInputStream();

    BufferedReader br = new BufferedReader(new InputStreamReader(in));
    String line = br.readLine();
    while(line!=null) {
       System.out.println(line);
       line = br.readLine();
    }    
}
```

命令行中使用/c关闭执行完毕的窗口，否则无法获取输入流。但是在Linux下面就可以直接使用如下代码获取输入流：
```java
Process process = Runtime.getRuntime().exec("ls");
int status = process.waitFor();
 
System.out.println(status);
InputStream in = process.getInputStream();
 
BufferedReader br = new BufferedReader(new InputStreamReader(in));
String line = br.readLine();
while(line!=null) {
    System.out.println(line);
    line = br.readLine();
}
```

还可以使用ProcessBuilder进行创建进程，这种方更灵活。代码如下：
```java
public static void processBuilderCommand() throws Exception {
    List<String> commands = new ArrayList<>();
    commands.add("cmd.exe");
    commands.add("/c");
    commands.add("dir");
    commands.add("E:\\flink");
    ProcessBuilder pb =new ProcessBuilder(commands);
    //可以修改进程环境变量
    pb.environment().put("DAXIN_HOME", "/home/daxin");
    System.out.println(pb.directory());
     
    Process process = pb.start();
    int status = process.waitFor();
    System.out.println(pb.environment());
     
    System.out.println(status);
    InputStream in = process.getInputStream();
     
    BufferedReader br = new BufferedReader(new InputStreamReader(in));
    String line = br.readLine();
    while(line!=null) {
        System.out.println(line);
        line = br.readLine();
    }
     
}
```

调用java命令执行class文件并获取输出：
```java
public static void processBuilderCommand() throws Exception {
   List<String> commands = new ArrayList<>();
   commands.add("cmd.exe");
   commands.add("/c");
   commands.add("java HelloWorld");
   ProcessBuilder pb =new ProcessBuilder(commands);
   pb.directory(new File("C:\\Users\\liuguangxin\\oxygen--workspace\\java8\\bin\\"));//设置工作目录
   Process process = pb.start();
   int status = process.waitFor();
    
   InputStream in = process.getInputStream();
    
   BufferedReader br = new BufferedReader(new InputStreamReader(in));
   String line = br.readLine();
   while(line!=null) {
       System.out.println(line);
       line = br.readLine();
   }
    
}
```
`java   C:\\Users\\liuguangxin\\oxygen--workspace\\java8\\bin\\HelloWorld`会将目录作为类名一起解析故无法执行。

