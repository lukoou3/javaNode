一般会在控制台上打印以下信息：
```
log4j:WARN No appenders could be found for logger (org.apache.hadoop.util.Shell).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```
然后不会出现MapReduce进度信息。

这种情况一般是由于log4j这个日志信息打印模块的配置信息没有给出造成的，可以在项目的src目录下，新建一个文件，命名为“log4j.properties”，填入以下信息：
```
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

就可以将Hadoop运行过程中产生的所有的以[INFO]开头的日志信息打印出来。
添加后，效果如下：
```
2019-04-23 22:47:21,440 INFO [org.apache.hadoop.conf.Configuration.deprecation] - session.id is deprecated. Instead, use dfs.metrics.session-id
2019-04-23 22:47:21,441 INFO [org.apache.hadoop.metrics.jvm.JvmMetrics] - Initializing JVM Metrics with processName=JobTracker, sessionId=
2019-04-23 22:47:22,182 WARN [org.apache.hadoop.mapreduce.JobResourceUploader] - Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2019-04-23 22:47:22,183 WARN [org.apache.hadoop.mapreduce.JobResourceUploader] - No job jar file set.  User classes may not be found. See Job or Job#setJar(String).
2019-04-23 22:47:22,344 INFO [org.apache.hadoop.mapreduce.lib.input.FileInputFormat] - Total input paths to process : 5
2019-04-23 22:47:22,359 INFO [org.apache.hadoop.mapreduce.lib.input.CombineFileInputFormat] - DEBUG: Terminated node allocation with : CompletedNodes: 1, size left: 355
2019-04-23 22:47:22,402 INFO [org.apache.hadoop.mapreduce.JobSubmitter] - number of splits:1
2019-04-23 22:47:22,459 INFO [org.apache.hadoop.mapreduce.JobSubmitter] - Submitting tokens for job: job_local1765482257_0001
2019-04-23 22:47:22,576 INFO [org.apache.hadoop.mapreduce.Job] - The url to track the job: http://localhost:8080/
2019-04-23 22:47:22,577 INFO [org.apache.hadoop.mapreduce.Job] - Running job: job_local1765482257_0001
2019-04-23 22:47:22,578 INFO [org.apache.hadoop.mapred.LocalJobRunner] - OutputCommitter set in config null
2019-04-23 22:47:22,584 INFO [org.apache.hadoop.mapred.LocalJobRunner] - OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
2019-04-23 22:47:22,628 INFO [org.apache.hadoop.mapred.LocalJobRunner] - Waiting for map tasks
2019-04-23 22:47:22,629 INFO [org.apache.hadoop.mapred.LocalJobRunner] - Starting task: attempt_local1765482257_0001_m_000000_0
2019-04-23 22:47:22,656 INFO [org.apache.hadoop.yarn.util.ProcfsBasedProcessTree] - ProcfsBasedProcessTree currently is supported only on Linux.
2019-04-23 22:47:22,690 INFO [org.apache.hadoop.mapred.Task] -  Using ResourceCalculatorProcessTree : org.apache.hadoop.yarn.util.WindowsBasedProcessTree@4fd741fe
2019-04-23 22:47:22,693 INFO [org.apache.hadoop.mapred.MapTask] - Processing split: Paths:/F:/hadoop_test/wordcount/input/a.txt:0+71,/F:/hadoop_test/wordcount/input/b.txt:0+71,/F:/hadoop_test/wordcount/input/c.txt:0+71,/F:/hadoop_test/wordcount/input/d.txt:0+71,/F:/hadoop_test/wordcount/input/e.txt:0+71
2019-04-23 22:47:22,736 INFO [org.apache.hadoop.mapred.MapTask] - (EQUATOR) 0 kvi 26214396(104857584)
2019-04-23 22:47:22,736 INFO [org.apache.hadoop.mapred.MapTask] - mapreduce.task.io.sort.mb: 100
2019-04-23 22:47:22,736 INFO [org.apache.hadoop.mapred.MapTask] - soft limit at 83886080
2019-04-23 22:47:22,736 INFO [org.apache.hadoop.mapred.MapTask] - bufstart = 0; bufvoid = 104857600
2019-04-23 22:47:22,736 INFO [org.apache.hadoop.mapred.MapTask] - kvstart = 26214396; length = 6553600
2019-04-23 22:47:22,753 INFO [org.apache.hadoop.mapred.MapTask] - Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2019-04-23 22:47:22,762 INFO [org.apache.hadoop.mapred.LocalJobRunner] - 
2019-04-23 22:47:22,762 INFO [org.apache.hadoop.mapred.MapTask] - Starting flush of map output
2019-04-23 22:47:22,762 INFO [org.apache.hadoop.mapred.MapTask] - Spilling map output
2019-04-23 22:47:22,762 INFO [org.apache.hadoop.mapred.MapTask] - bufstart = 0; bufend = 605; bufvoid = 104857600
2019-04-23 22:47:22,762 INFO [org.apache.hadoop.mapred.MapTask] - kvstart = 26214396(104857584); kvend = 26214140(104856560); length = 257/6553600
2019-04-23 22:47:22,775 INFO [org.apache.hadoop.mapred.MapTask] - Finished spill 0
2019-04-23 22:47:22,784 INFO [org.apache.hadoop.mapred.Task] - Task:attempt_local1765482257_0001_m_000000_0 is done. And is in the process of committing
2019-04-23 22:47:22,790 INFO [org.apache.hadoop.mapred.LocalJobRunner] - map
2019-04-23 22:47:22,790 INFO [org.apache.hadoop.mapred.Task] - Task 'attempt_local1765482257_0001_m_000000_0' done.
2019-04-23 22:47:22,790 INFO [org.apache.hadoop.mapred.LocalJobRunner] - Finishing task: attempt_local1765482257_0001_m_000000_0
2019-04-23 22:47:22,790 INFO [org.apache.hadoop.mapred.LocalJobRunner] - map task executor complete.
2019-04-23 22:47:22,792 INFO [org.apache.hadoop.mapred.LocalJobRunner] - Waiting for reduce tasks
2019-04-23 22:47:22,792 INFO [org.apache.hadoop.mapred.LocalJobRunner] - Starting task: attempt_local1765482257_0001_r_000000_0
2019-04-23 22:47:22,798 INFO [org.apache.hadoop.yarn.util.ProcfsBasedProcessTree] - ProcfsBasedProcessTree currently is supported only on Linux.
2019-04-23 22:47:22,827 INFO [org.apache.hadoop.mapred.Task] -  Using ResourceCalculatorProcessTree : org.apache.hadoop.yarn.util.WindowsBasedProcessTree@3db89990
2019-04-23 22:47:22,831 INFO [org.apache.hadoop.mapred.ReduceTask] - Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@9b43490
2019-04-23 22:47:22,840 INFO [org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl] - MergerManager: memoryLimit=2653054464, maxSingleShuffleLimit=663263616, mergeThreshold=1751016064, ioSortFactor=10, memToMemMergeOutputsThreshold=10
2019-04-23 22:47:22,841 INFO [org.apache.hadoop.mapreduce.task.reduce.EventFetcher] - attempt_local1765482257_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
2019-04-23 22:47:22,864 INFO [org.apache.hadoop.mapreduce.task.reduce.LocalFetcher] - localfetcher#1 about to shuffle output of map attempt_local1765482257_0001_m_000000_0 decomp: 94 len: 98 to MEMORY
2019-04-23 22:47:22,869 INFO [org.apache.hadoop.mapreduce.task.reduce.InMemoryMapOutput] - Read 94 bytes from map-output for attempt_local1765482257_0001_m_000000_0
2019-04-23 22:47:22,870 INFO [org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl] - closeInMemoryFile -> map-output of size: 94, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->94
2019-04-23 22:47:22,871 INFO [org.apache.hadoop.mapreduce.task.reduce.EventFetcher] - EventFetcher is interrupted.. Returning
2019-04-23 22:47:22,872 INFO [org.apache.hadoop.mapred.LocalJobRunner] - 1 / 1 copied.
2019-04-23 22:47:22,879 INFO [org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl] - finalMerge called with 1 in-memory map-outputs and 0 on-disk map-outputs
2019-04-23 22:47:22,886 INFO [org.apache.hadoop.mapred.Merger] - Merging 1 sorted segments
2019-04-23 22:47:22,886 INFO [org.apache.hadoop.mapred.Merger] - Down to the last merge-pass, with 1 segments left of total size: 88 bytes
2019-04-23 22:47:22,889 INFO [org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl] - Merged 1 segments, 94 bytes to disk to satisfy reduce memory limit
2019-04-23 22:47:22,889 INFO [org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl] - Merging 1 files, 98 bytes from disk
2019-04-23 22:47:22,890 INFO [org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl] - Merging 0 segments, 0 bytes from memory into reduce
2019-04-23 22:47:22,890 INFO [org.apache.hadoop.mapred.Merger] - Merging 1 sorted segments
2019-04-23 22:47:22,891 INFO [org.apache.hadoop.mapred.Merger] - Down to the last merge-pass, with 1 segments left of total size: 88 bytes
2019-04-23 22:47:22,891 INFO [org.apache.hadoop.mapred.LocalJobRunner] - 1 / 1 copied.
2019-04-23 22:47:22,894 INFO [org.apache.hadoop.conf.Configuration.deprecation] - mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
2019-04-23 22:47:22,902 INFO [org.apache.hadoop.mapred.Task] - Task:attempt_local1765482257_0001_r_000000_0 is done. And is in the process of committing
2019-04-23 22:47:22,903 INFO [org.apache.hadoop.mapred.LocalJobRunner] - 1 / 1 copied.
2019-04-23 22:47:22,903 INFO [org.apache.hadoop.mapred.Task] - Task attempt_local1765482257_0001_r_000000_0 is allowed to commit now
2019-04-23 22:47:22,904 INFO [org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter] - Saved output of task 'attempt_local1765482257_0001_r_000000_0' to file:/F:/hadoop_test/wordcount/output/_temporary/0/task_local1765482257_0001_r_000000
2019-04-23 22:47:22,904 INFO [org.apache.hadoop.mapred.LocalJobRunner] - reduce > reduce
2019-04-23 22:47:22,904 INFO [org.apache.hadoop.mapred.Task] - Task 'attempt_local1765482257_0001_r_000000_0' done.
2019-04-23 22:47:22,904 INFO [org.apache.hadoop.mapred.LocalJobRunner] - Finishing task: attempt_local1765482257_0001_r_000000_0
2019-04-23 22:47:22,905 INFO [org.apache.hadoop.mapred.LocalJobRunner] - reduce task executor complete.
2019-04-23 22:47:23,580 INFO [org.apache.hadoop.mapreduce.Job] - Job job_local1765482257_0001 running in uber mode : false
2019-04-23 22:47:23,581 INFO [org.apache.hadoop.mapreduce.Job] -  map 100% reduce 100%
2019-04-23 22:47:23,581 INFO [org.apache.hadoop.mapreduce.Job] - Job job_local1765482257_0001 completed successfully
2019-04-23 22:47:23,588 INFO [org.apache.hadoop.mapreduce.Job] - Counters: 33
	File System Counters
		FILE: Number of bytes read=1770
		FILE: Number of bytes written=514613
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=20
		Map output records=65
		Map output bytes=605
		Map output materialized bytes=98
		Input split bytes=370
		Combine input records=65
		Combine output records=8
		Reduce input groups=8
		Reduce shuffle bytes=98
		Reduce input records=8
		Reduce output records=8
		Spilled Records=16
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=7
		CPU time spent (ms)=0
		Physical memory (bytes) snapshot=0
		Virtual memory (bytes) snapshot=0
		Total committed heap usage (bytes)=514850816
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=0
	File Output Format Counters 
		Bytes Written=75

```