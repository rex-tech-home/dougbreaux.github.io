---
title: wsadmin Script to check the JVM heap size of a WebSphere Server
tags: [script,jvm,websphere,heap,jython,wsadmin]
---
In response to [this StackOverflow question](http://stackoverflow.com/questions/8754185/how-to-lookup-heap-usage-for-websphere), here's [a simple Jython wsadmin script](https://github.com/dougbreaux/websphere/blob/master/getJvmHeap.py) which will display the JVM min and max heap sizes for the specified Application Server.

```shell
# Usage: wsadmin -f getJvmHeap.py server  
server = AdminConfig.getid('/Server:'+sys.argv[0]+'/')  
jvm = AdminConfig.list('JavaVirtualMachine', server)  

print 'initialHeapSize: ' + AdminConfig.showAttribute(jvm, 'initialHeapSize')  
print 'maximumHeapSize: ' + AdminConfig.showAttribute(jvm, 'maximumHeapSize')
```

**Update**: the above only obtains the heap size if a custom value has been explicitly configured. From [this other post]({% post_url 2012-08-24-misc-wsadmin%}), here's a mechanism which will obtain the maximum heap size otherwise (note there doesn't appear to be an attribute to obtain the minimum heap size):

```shell
jvmName = AdminControl.completeObjectName('WebSphere:type=JVM,process=MyServerName,*')  
print AdminControl.getAttribute(jvmName,'maxMemory')
```
