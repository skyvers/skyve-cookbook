# Skyve Cookbook
Examples and code samples for using the [Skyve](http://skyve.org/) framework.

### Contents
* [Deproxy](#deproxy)
* [Ordering a collection in a data table](#ordering-a-collection-in-a-data-table)
* [Troubleshooting](#troubleshooting)
  * [Heap space errors](heap-space-errors)
* [Creating Rest Services](#creating-rest-services)

### Deproxy 
When to use:
- performing an `instanceof` fails when operating over subclasses of a document with a persistence strategy of single
- Skyve logs an error performing a `BindUtil.get()`:
  
```
14:09:22,785 SEVERE [SKYVE] (default task-14) Could not BindUtil.get(modules.finance.Transaction.TransactionExtension@b46ca56a#97175c28-e505-4f39-911e-9c6e11d0d6bf, allocations[0].relatedTransaction.source.selectedDebitAllocation)!
14:09:22,785 SEVERE [SKYVE] (default task-14) The subsequent stack trace relates to obtaining bean property selectedDebitAllocation from modules.finance.domain.PaymentIn@de1d73c9#58db72b4-4ad1-451d-9526-29ed7112f1d9
```

The `Util.deproxy()` utility method lets you resolve this by loading the correct subclass.
```java
Occurrence occ = edc.getToBeOccurrence();
occ = Util.deproxy(occ);
if (occ instanceof DeviceOccurrence) {
}
```

**[⬆ back to top](#contents)**

### Ordering a collection in a data table
Collections can be ordered or unordered, where an ordered collection allows the user to define the sort of items in the list via drag and drop. Ordering on a collection can be enabled by adding the `ordered="true"` attribute to the collection declaration.

Collections can also have a default sorting by specifying the sort columns in the collection declaration in the document:
```xml
<ordering>
  <order by="binding1" sort="ascending" />
  <order by="binding2" sort="ascending" />
</ordering>
```

**[⬆ back to top](#contents)**

### Troubleshooting

#### Heap space errors
During startup of the server, if you encounter a `java.lang.OutOfMemoryError: Java heap space`, a possible cause could be an error in your json file which can't be parse correctly. A sample stacktrace of this is below:
```
14:12:51,910 ERROR [org.jboss.msc.service.fail] (ServerService Thread Pool -- 64) MSC000001: Failed to start service jboss.undertow.deployment.default-server.default-host./skyve: org.jboss.msc.service.StartException in service jboss.undertow.deployment.default-server.default-host./skyve: java.lang.OutOfMemoryError: Java heap space
    at org.wildfly.extension.undertow.deployment.UndertowDeploymentService$1.run(UndertowDeploymentService.java:85)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
    at org.jboss.threads.JBossThread.run(JBossThread.java:320)
Caused by: java.lang.OutOfMemoryError: Java heap space
    at java.util.Arrays.copyOf(Arrays.java:3332)
    at java.lang.AbstractStringBuilder.expandCapacity(AbstractStringBuilder.java:137)
    at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:121)
    at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:622)
    at java.lang.StringBuffer.append(StringBuffer.java:383)
    at org.skyve.impl.util.json.JSONReader.add(JSONReader.java:421)
    at org.skyve.impl.util.json.JSONReader.add(JSONReader.java:426)
    at org.skyve.impl.util.json.JSONReader.string(JSONReader.java:410)
    at org.skyve.impl.util.json.JSONReader.read(JSONReader.java:165)
    at org.skyve.impl.util.json.JSONReader.object(JSONReader.java:293)
    at org.skyve.impl.util.json.JSONReader.read(JSONReader.java:103)
    at org.skyve.impl.util.json.JSONReader.read(JSONReader.java:79)
    at org.skyve.util.JSON.unmarshall(JSON.java:35)
    at org.skyve.impl.util.UtilImpl.readJSONConfig(UtilImpl.java:197)
    at org.skyve.impl.web.SkyveContextListener.contextInitialized(SkyveContextListener.java:60)
```
**[⬆ back to top](#contents)**

### Creating REST Services

The `BasicAuthFilter` provided in Skyve can be used to allow authentication using a HTTP basic auth header. So a user can make REST requests using their existing credentials, or create a specific API user in Skyve which has permission to the API.

The `web.xml` in the project will need to be updated to configure the filter, similar to the SkyveFacesFilter. Specify the 
init params for realm (optional and defaulted) and unsecured (optional).

You can either configure the filter in `web.xml` (see SkyveFacesFilter for something simliar) or by using a `@WebFilter` annotation on the class.

For example: 

```xml
@WebFilter(filterName = "BasicAuthFilter", urlPatterns = {"/api/*"})

```
The filter has to init parameters:

```java
private String realm = “Protected”;
private String[] unsecuredURLPrefixes;
```

The realm is used when an unauthorised response is sent, it is an arbitrary value.
The unsecured URL prefixes allows you to create exceptions that will not be secured by the filter.
Its similar in use to SkyveFacesFilter in web.xml (the URL prefixes are separated by a newline).
