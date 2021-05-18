# JVM OOM file storage solution

OOM-kill is out-of-memory, there is a layer of protection mechanism in linux kernel, used to avoid linux from serious problems when out of memory, kill the irrelevant processes.

However, there are times when we need to troubleshoot the problem after our application is killed by the OOM killer. We can analyze dump logs to locate and analyze OOM exceptions.

## How to capture Dump files？

This is achieved by setting JVM parameters.

### Related parameters

- `-XX:+HeapDumpOnOutOfMemoryError`

	Set to export information about the heap at this time when a memory overflow is first encountered.

- `-XX:HeapDumpPath`

	Specify the path or file name when exporting heap information, e.g.：`-XX:HeapDumpPath=/tmp/heapdump.hprof`
	
	
### Specific use

- Set Jvm parameters when starting via Java command：

```bash
java -XX:+PrintFlagsFinal -XX:+UnlockExperimentalVMOptions \
	-XX:+UseCGroupMemoryLimitForHeap \
	-XX:+HeapDumpOnOutOfMemoryError \
	-XX:HeapDumpPath=/tmp/heapdump.hprof -jar app.jar
```

- If deploying Java applications using containers:

```bash
docker run \
-e JAVA_OPTS="... -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof..." \
......
```

- Also, specify that the container platform collects the `/tmp/heapdump.hprof` file from the container.
