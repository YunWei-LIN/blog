title: ClientAbortException
date: 2016-2-15 16:10:08
tags: [tomcat,Spring MVC]  
categories: exception
---
Spring MVC 4.2.1.RELEASE ， 容器 tomcat 8.21， nginx ： 客户端在服务端响应前就关闭（socket），比如下载文件一半关闭浏览器，
服务端会有如下异常， 该异常服务器从业务上不需要处理。
```
    org.apache.catalina.connector.ClientAbortException: java.io.IOException: 断开的管道
	    at org.apache.catalina.connector.OutputBuffer.realWriteBytes(OutputBuffer.java:393) ~[catalina.jar:8.0.18]
	    at org.apache.tomcat.util.buf.ByteChunk.flushBuffer(ByteChunk.java:426) ~[tomcat-util.jar:8.0.18]
	    at org.apache.catalina.connector.OutputBuffer.doFlush(OutputBuffer.java:342) ~[catalina.jar:8.0.18]
	    at org.apache.catalina.connector.OutputBuffer.flush(OutputBuffer.java:317) ~[catalina.jar:8.0.18]
	    at org.apache.catalina.connector.CoyoteOutputStream.flush(CoyoteOutputStream.java:110) ~[catalina.jar:8.0.18]
	    ......
	    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:516) [catalina.jar:8.0.18]
	    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1086) [tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:659) [tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223) [tomcat-coyote.jar:8.0.18]
	    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1558) [tomcat-coyote.jar:8.0.18]
	    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1515) [tomcat-coyote.jar:8.0.18]
	    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [na:1.8.0_31]
	    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [na:1.8.0_31]
	    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-util.jar:8.0.18]
	    at java.lang.Thread.run(Thread.java:745) [na:1.8.0_31]
    Caused by: java.io.IOException: 断开的管道
	    at sun.nio.ch.FileDispatcherImpl.write0(Native Method) ~[na:1.8.0_31]
	    at sun.nio.ch.SocketDispatcher.write(SocketDispatcher.java:47) ~[na:1.8.0_31]
	    at sun.nio.ch.IOUtil.writeFromNativeBuffer(IOUtil.java:93) ~[na:1.8.0_31]
	    at sun.nio.ch.IOUtil.write(IOUtil.java:65) ~[na:1.8.0_31]
	    at sun.nio.ch.SocketChannelImpl.write(SocketChannelImpl.java:470) ~[na:1.8.0_31]
	    at org.apache.tomcat.util.net.NioChannel.write(NioChannel.java:127) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.tomcat.util.net.NioBlockingSelector.write(NioBlockingSelector.java:101) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.tomcat.util.net.NioSelectorPool.write(NioSelectorPool.java:173) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.http11.InternalNioOutputBuffer.writeToSocket(InternalNioOutputBuffer.java:139) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.http11.InternalNioOutputBuffer.addToBB(InternalNioOutputBuffer.java:197) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.http11.InternalNioOutputBuffer.access$000(InternalNioOutputBuffer.java:41) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.http11.InternalNioOutputBuffer$SocketOutputBuffer.doWrite(InternalNioOutputBuffer.java:320) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.http11.filters.IdentityOutputFilter.doWrite(IdentityOutputFilter.java:93) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.http11.AbstractOutputBuffer.doWrite(AbstractOutputBuffer.java:256) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.coyote.Response.doWrite(Response.java:503) ~[tomcat-coyote.jar:8.0.18]
	    at org.apache.catalina.connector.OutputBuffer.realWriteBytes(OutputBuffer.java:388) ~[catalina.jar:8.0.18]
	    ... 76 common frames omitted
```
