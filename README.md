# Skyve Cookbook
Examples and code samples for using the [Skyve](http://skyve.org/) framework.

### Contents
* [Skyve Script](#skyve-script)
* [Deproxy](#deproxy)
* [Ordering a collection in a data table](#ordering-a-collection-in-a-data-table)
* [Troubleshooting](#troubleshooting)
  * [Heap space errors](heap-space-errors)
* [Creating Rest Services](#creating-rest-services)
* [Understanding Skyve Rest](#understanding-skyve-rest)
* [Problems with utf8 - character sets for other languages - MySQL](#problems-with-utf8---character-sets-for-other-languages---mysql)
* [Customer Scoped Roles](#customer-scoped-roles)
* [SAIL Automated UI Tests](#sail-automated-ui-tests)
* [Setting up a Skyve instance](#setting-up-a-skyve-instance)
* [Manually seeding new customer users](#manually-seeding-new-customer-users)

### Skyve Script
Skyve Script is a new abbreviated way to declare a no-code application - using the markdown standard to allow developers to specify domain models.

#### Quick Overview
In Skyve Script, a document declaration looks like this:
```markdown
## Address `MDL_Address`
- *addressLine1* text 100
- addressLine2 text 100
- addressLine3 text 100
- *suburb* text 50
- state enum (QLD,NSW,WA,NT,ACT,SA,VIC,TAS)
- *country* Country
- verified boolean
```

Skyve Script supports complex applications, with relationships between documents, for example:
```markdown
## Organisation `MDL_Organisation`
- *name* text 100
- businessAddress Address
- postalAddress Address
- deliveryAddress Address
```

The script for a complete 'Organisation Address Book' application module would then be:
```markdown
# "Organisation Address Book"

## Address `MDL_Address`
- *addressLine1* text 100
- addressLine2 text 100
- addressLine3 text 100
- *suburb* text 50
- state enum (QLD,NSW,WA,NT,ACT,SA,VIC,TAS)
- *country* Country
- verified boolean

## Country `MDL_Country`
- *name* text 100

## Organisation `MDL_Organisation`
- *name* text 100
- businessAddress Address
- postalAddress Address
- deliveryAddress Address
```

#### Using Skyve Script
You can use Skyve Script with the Skyve online project creator for a new project, or within an existing Skyve application to create additional documents.

Within an existing Skyve application, the Skyve admin module provides the Document Creator - a user interface for editing Skyve Script. By placing your markdown in the Input tab, the system will preview the markdown and generated document declaration when you change tab. Any errors will be highlighted in red in the Document Preview tab.

The power of Skyve Script is that you can create it anywhere anytime, on your phone, in notepad or on paper. You don't need a UML modelling application or diagramming tool, but it is sufficient to create a functioning no-code application in Skyve.

While Skyve Script is sophisticated enough to build real no-code applications, it is intended as a rapid development technique and not a replacement for a complete Skyve declaration. While the full Skyve declaration standard supports rich domain models sufficient for enterprise scale, sophisticated mission critical systems, Skyve Script will get you to a functioning no-code application in minutes.

#### Syntax
Skyve Script has a few specific syntax requirements it is looking for when generating domain models:

* New modules should be specified with a level 1 heading (#), followed by a space, then the module name in title case
  * e.g. `# Admin`
  * If a specific module title is required, it can be encapsulated in single or double quotes, e.g. `# 'My Module'`
* New domain models (documents), should be specified with a level 2 heading (##), followed by a space, then the document name in title case
  * e.g. `## User`
  * If a specific document title is required, it can be encapsulated in single or double quotes, e.g. `## 'My Document'`
  * If the document is to be persisted, the persistent name should follow the document name surrounded by backticks (&#96;), e.g. `` ## User `ADM_User` ``
* Domain attributes (fields) should be specified as a list below the domain heading, by using a dash (-), followed by a space, then the attribute definition
  * Attribute definitions should start with the attribute name in camel case (no spaces, first letter lowercase, uppercase for new words), e.g. `firstName`
  * Required attributes should have their name surrounded by asterisks, e.g. `- *firstName*`
  * The type should follow the attribute name
  * If the type is text, the length should follow the type, e.g. `- *firstName* text 150`
  * If the type is enum, the values should follow the type in brackets, e.g. `- state enum ("Not Started", 'In Progress', Complete)`
  * If a specific display name is required, it can be encapsulated in single or double quotes, e.g. `- 'Yes/No'`
* Associations should be specified as a list item by using a dash (-), followed by a space, then the association name in camel case, then the association document in title case
  * e.g. `- country Country`
  * Required associations should have their name surrounded by asterisks, e.g. `- *country* Country`
  * Association type can be specified surrounded by backticks after the document name, e.g. `` - country Country `composition` ``
* Collections should be specified as a list item by using a plus (+), followed by a space, then the collection name in camel case, then the collection document in title case
  * e.g. `+ roles Role`
  * Required collections should have their name surrounded by asterisks, e.g. `+ *roles* Role`
  * Collection type can be specified surrounded by backticks after the document name, e.g. `` + roles Role `child` ``

**[⬆ back to top](#contents)**

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
private String realm = "Protected";
private String[] unsecuredURLPrefixes;
```

The realm is used when an unauthorised response is sent, it is an arbitrary value.
The unsecured URL prefixes allows you to create exceptions that will not be secured by the filter.
Its similar in use to SkyveFacesFilter in web.xml (the URL prefixes are separated by a newline).

### Understanding Skyve Rest 
Skyve provides a number of Rest endpoints to support whatever application you build.

However to use these you will require a knowledge of Rest and web filters. An integration test is available at [BasicAuthIT.java](https://github.com/skyvers/skyve/blob/master/skyve-ee/src/test/org/skyve/impl/web/filter/rest/BasicAuthIT.java).

You can  trial the endpoints in your browser to gain an understanding of how they work, before you begin coding interactions. To trial the endpoints, you'll need to edit the web.xml which is located in your project at:
`/src/main/webapp/WEB-INF/web.xml`

#### Configuring The Session Filter
Insert this filter and mapping after the other filters:
```xml
<filter>
  <display-name>RestFilter</display-name>
  <filter-name>RestFilter</filter-name>
  <filter-class>org.skyve.impl.web.filter.rest.SessionFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>RestFilter</filter-name>
  <url-pattern>/rest/*</url-pattern>
</filter-mapping>
```    

Then redploy your app (or restart your app server).

#### Testing end points
The SessionFilter will allow you to interact with the end points while you have a valid Session, and you'll be redirected to a login page at the first interaction to authenticate. Once you've logged in, you can then exercise the endpoints using your browser. The SessionFilter allows you to make REST calls after initial login that will propagate the remote user onto the REST call's execution context (the logged in user).

For example:

```
http://localhost:8080/<projectName>/rest/json/admin/Contact
```

This end point will return an array of json strings for all user contacts. Note that the address includes
- _rest_ - the filtered service 
- _json_ - the specific implemented result type
- _admin_ - the Skyve module
- _Contact_ - the Skyve document

Your result will be similar to the following:
```json
[
  {
    "bizModule":"admin",
    "bizDocument":"Contact",
    "name":"Aaliyah Bowling",
    "contactType":"Person",
    "email1":"aaliyah.bowling@whosin.com",
    "mobile":"0474 618 810",
    "image":"",
    "bizId":"58619c0d-496b-49e6-9fcb-ff43281ae740",
    "bizCustomer":"demo",
    "bizDataGroupId":null,
    "bizUserId":"bf8cb7c4-5d8a-477b-8b96-15a2bc72364a"
  }
]
```

In the example above, there is only one Contact in the database, called admin. The fields returned are the fields which would be returned if you used the menu item Contacts in admin module, that is, the default Skyve query for Contacts.

To retrieve a specific instance, use the bizId of the instance in the address, for example:

```
http://localhost:8080/<projectName>/rest/json/admin/Contact/58619c0d-496b-49e6-9fcb-ff43281ae740
```

Note that the id matches the bizId in the prior example, and that the entire Contact object is now retrieved - not only the fields in the default query. This includes the result of conditions declared in the Contact document.

```json
 {
  "bizModule":"admin",
  "bizDocument":"Contact",
  "name":"Aaliyah Bowling",
  "contactType":"Person",
  "email1":"aaliyah.bowling@whosin.com",
  "mobile":"0474 618 810",
  "image":"",
  "bizId":"58619c0d-496b-49e6-9fcb-ff43281ae740",
  "bizCustomer":"demo",
  "bizDataGroupId":null,
  "bizUserId":"bf8cb7c4-5d8a-477b-8b96-15a2bc72364a",
  "bizVersion":5,
  "bizLock":"20170410143341392demo"
 }
```
Note the attributes, bizCustomer, bizModule and bizDocument define the specific entity. When using insert and update endpoints, these attributes specify the entity to be manipulated.


To retrieve data from a specific query, provide the query name, for example:

```
http://localhost:8080/<projectName>/rest/json/query/admin/qContacts/
```

To retrieve a paged result set, provide the start and end rows and the specific Skyve query name, for example to retrieve results 0 to 9:

```
http://localhost:8080/siteStorage/rest/json/query/admin/qContacts?start=0&end=9
```

There are also endpoints provide for insert, update and delete. 

```
http://localhost:8080/<projectName>/rest/json/insert/
http://localhost:8080/<projectName>/rest/json/update/
http://localhost:8080/<projectName>/rest/json/delete/
```

These require a complete json representation of the object to be manipulated. These endpoints utilise the bizCustomer, bizModule and bizDocument within the json representation.

For example, the update the email address of the contact `admin` in the prior example from `aaliyah.bowling@whosin.com` to `aaliyah.bowling@skyve.org`, supply the modified json as follows:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"bizModule":"admin","bizDocument":"Contact","name":"Aaliyah Bowling","contactType":"Person","email1":"aaliyah.bowling@skyve.org","mobile":"0474 618 810","image":"","bizId":"58619c0d-496b-49e6-9fcb-ff43281ae740","bizCustomer":"demo","bizDataGroupId":null,"bizUserId":"bf8cb7c4-5d8a-477b-8b96-15a2bc72364a","bizVersion":5,"bizLock":"20170410143341392demo"}' http://localhost:8080/<projectName>/rest/json/update/
```
_Note: on Windows, you will need to use double quotes instead of single quotes around the JSON, and escape the double quotes with a backslash, e.g. "{\"created\":true. This can also be performed as a GET request if the JSON is properly escaped._

#### Using the BasicAuthFilter
When you're ready to start coding comment out the following in your `web.xml`:

```xml
<!-- Secure rest services -->
<url-pattern>/rest/*</url-pattern>
```

This will stop these url patterns redirecting to the login page, and change the filter to the `BasicAuthFilter.java` as follows:

```xml
<filter-class>org.skyve.impl.web.filter.rest.SessionFilter</filter-class>
```

to

```xml
<filter-class>org.skyve.impl.web.filter.rest.BasicAuthFilter</filter-class>
```

The BasicAuthFilter allows authentication using a HTTP basic auth header. You can use the class by just changing the package to your package - there are no extra dependencies introduced by its use.

You can either configure the filter in web.xml (see SkyveFacesFilter for something similar) or by using a `@WebFilter` annotation on the class.

For example:

```java
@WebFilter(filterName = "BasicAuthFilter", urlPatterns = {"/api/*"})
```

The filter has two init parameters:

```java
private String realm = "Protected";
private String[] unsecuredURLPrefixes;
```

The realm is used when an unauthorised response is sent (it is an arbitrary value). The unsecured URL prefixes allows you to create exceptions that will not be secured by the filter. For comparison, review the SkyveFacesFilter in web.xml (the URL prefixes are separated by a newline).

#### Example (CURL)
```bash
curl --user name:password http://192.168.43.91:8080/<projectName>/rest/json/query/admin/qContacts
```

#### Example (React)
```javascript
const base64 = require('base-64');
var headers = new Headers();
headers.append("Authorization", "Basic " + base64.encode("admin:password01"));
console.log(headers);
fetch('http://192.168.43.91:8080/<projectName>/rest/json/query/admin/qContacts', {
headers: headers
})
.then((response) => {
console.log('response:',response);
})
.catch((error) => {
console.error("err" + error);
}
```

#### Using the RestUserPersistenceFilter
If your REST API:

- does not use basic authentication
- requires some other form of authentication not able to be performed in a filter
- is not being performed on behalf of a system user (e.g. a request from a 3rd party service)

You will still need a user set on the Skyve Persistence in order to query or update the database. The BasicAuthFilter attempst to use the Basic credentials supplied to authenticate and perform database requests. Another option is to create a specific API user which has the roles and permissions the API can access. The RestUserPersistenceFilter allows a named user to be specified for all requests which match a URL pattern.

If this sounds like the scenario you require, to begin using the filter addo the following in your `web.xml` (near the other filters):

```xml
<filter>
	<filter-name>RestUserPersistenceFilter</filter-name>
	<filter-class>org.skyve.impl.web.filter.rest.RestUserPersistenceFilter</filter-class>
	<init-param>
		<param-name>PersistenceUser</param-name>
		<param-value>{restUsername}</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>RestUserPersistenceFilter</filter-name>
	<url-pattern>{restUrlPattern}</url-pattern>
</filter-mapping>
```

You will need to change `{restUsername}` to the username of the API user you create in the admin module.

Modify `{restUrlPattern}` to be the url endpoint your API controller is listening at.

You will also need to update `src/main/webapp/WEB-INF/spring/security.xml` to allow unauthenticated requests:

```xml
<intercept-url pattern="{restUrlPattern}" access="permitAll" method="POST" />
```

#### Other Resources
https://github.com/skyvers/skyve/tree/master/skyve-web/src/main/java/org/skyve/impl/web/service/rest
https://github.com/skyvers/skyve/blob/master/skyve-ee/src/test/org/skyve/impl/web/filter/rest/BasicAuthIT.java

**[⬆ back to top](#contents)**

### Problems with utf8 - character sets for other languages - MySQL
If your Skyve application is not storing utf8 chars correctly, and you're using MySQL, check that MySQL is configured for utf8. Check the charset of the DB and tables, e.g. the default  is 'latin1'.

In the my.cnf file (for MySQL), check that you have the following:
```
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

To keep an existing database once this change has been made, export the schema from MySQL workbench, use text edit change latin1 to utf8, then drop your schema and import the edited one.

If you don't need to keep existing data, then after the my.cnf changes above, drop your schema, create a new one, then use Skyve bootstrap (in the json settings file) to log in and let Skyve create the new schema for you.

#### Other Resources
https://stackoverflow.com/questions/3513773/change-mysql-default-character-set-to-utf-8-in-my-cnf

**[⬆ back to top](#contents)**

### Customer Scoped Roles

While Groups and Roles allow user permissions to be managed at the _module_ level, some applications have permissions which span multiple modules and the permissions need a way to be linked to function correctly (e.g. a user may have incomplete access to application features if module permissions were not correctly applied to their account). To address this, a customer scoped role can be defined in the `customer.xml` which allows roles to be aggregated together into something coherent and cross module.

```xml
<roles allowModuleRoles="true">
    <!-- External Access Roles -->
    <role name="External Basic">
        <description>Basic role for all external users</description>
        <roles>
            <role module="admin" name="BasicUser"/>
            <role module="admin" name="AppUser"/>
            <role module="module1" name="ExternalBasic"/>
            <role module="module2" name="ExternalBasic"/>
        </roles>
    </role>
</roles>
```

The `allowModuleRoles` property controls whether the module roles show up in the admin User Interface or not.

Customer roles aggregate only module roles, they cannot reference other customer roles. Groups can be used to aggregate either module rules or customer roles (if enabled by `allowModuleRoles`).

### SAIL Automated UI Tests
- download the latest chromdriver.exe from http://chromedriver.chromium.org/downloads
- update src/test/java/sail/admin/AdminSail.java with the location of the downloaded chromedriver (line 12) and the baseUrl (line 17)
- ensure you have the following dependencies in the project pom.xml
```
       <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-support</artifactId>
            <version>3.11.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.2</version>
            <scope>test</scope>
        </dependency>        
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-chrome-driver</artifactId>
            <version>3.11.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-firefox-driver</artifactId>
            <version>3.11.0</version>
            <scope>test</scope>
        </dependency>  
```
- generate domain to make sure dependencies have been downloaded
- check src/test/java/sail/admin/AdminFunctionSail.java and update the login instructions with correct login credentials for your project (login doesn't appear to support a default customer - so remove that from .json if you have it)
- start wildfly server and deploy project
- right-click AdminFunctionSail and run as JUnit- browser should fire up and tests should begin

### Setting up a Skyve instance

#### Recommended requirements 
We recommend the following:
- 4GB RAM for Linux and 8GB RAM for Windows
- Java JDK 8u191 (this is the JDK for Java 8)
- Wildfly Wildfly 10.1.0.Final
- Disk space requirements depend on the nature of the application especially if the database and content repository are located on the same drive, however, for most common applications, 50GB drive space will probably be sufficient.

#### Installation of prerequisites
To run a Skyve application, the server requires:

Java 8 (also called 1.8) - while the JRE is sufficient, the JDK is recommended.
 - Download the Java JDK 8u191 from https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 
 - These instructions may assist for linux - https://docs.oracle.com/javase/8/docs/technotes/guides/install/linux_jdk.html#BJFGGEFG (though note that this mentions an slightly older version of Java)

Wildfly 10.1.0.Final 
 - Download from http://wildfly.org/downloads/   
 - This link may assist - https://linuxtechlab.com/wildfly-10-10-1-0-installation/ 

#### Installing database driver
For database access, load the appropriate driver and declare this driver in the Wildfly standalone.xml configuration file.

For example, for SQL Server:
- load the sqljdbc42.jar into wildfy.../modules/system/layers/base/com/microsoft/sqlserver/main/
- copy the following definition into a new file  wildfy.../modules/system/layers/base/com/microsoft/sqlserver/main/module.xml
```
 		<?xml version="1.0" encoding="utf-8"?> 
			<module xmlns="urn:jboss:module:1.3" name="com.microsoft.sqlserver"> 
  			<resources> 
    				<resource-root path="sqljdbc42.jar"/> 
  			</resources> 
  			<dependencies> 
    				<module name="javax.api"/> 
    				<module name="javax.transaction.api"/>
				<module name="javax.xml.bind.api"/>
  			</dependencies> 
		</module>
```

- declare the driver in the wildfly configuration file wildfly/standalone/configuration/standalone.xml <drivers> stanza as follows:
```
		<driver name="sqlserver" module="com.microsoft.sqlserver">
                        <xa-datasource-class>com.microsoft.sqlserver.jdbc.SQLServerXADataSource</xa-datasource-class>
                </driver>
```

#### Configuring ports
To configure which ports will be used for accessing the application, modify the <socket-binding-group> section in the wildfly configuration file wildfly/standalone/configuration/standalone.xml for http and https:
```
        <socket-binding name="http" port="${jboss.http.port:8080}"/>
        <socket-binding name="https" port="${jboss.https.port:8443}"/>
````
For example, for external access, typically you would assign as follows:
```
	<socket-binding name="http" port="${jboss.http.port:80}"/>
        <socket-binding name="https" port="${jboss.https.port:443}"/>
```

#### Create a folder for content
Skyve includes the elastic content repository - the repository requires a dedicated folder to persist files. The user credential running wildfly (for example) will need read-write permissions to this folder.

#### Install the wildfly service
So that the Skyve application will be always available, install the wildfly service, ensuring that the service will have read/write access to the content folder.
The following may be useful for linux installations - https://community.i2b2.org/wiki/display/getstarted/2.4.2.3+Run+Wildfly+as+a+Linux+Service

#### Skyve application configuration
To deploy a Skyve application, there are typically three artefacts:
- the application '.war' folder
- the datasource 'ds.xml' file
- the properties '.json' file

For example, if your Skyve application is called 'helloworld', these will be:
- 'helloworld.war'
- 'helloworld-ds.xml'
- 'helloworld.json'

The 'ds.xml' and '.json' remain on the server you are deploying to, and are customised for that specific instance, so that when you are  deploying a new version of your Skyve application, the instance settings do not need to be adjusted.

When deploying the Skyve application web archive 'war', ensure that matching configuration settings are updated in the associated 'ds.xml' and '.json' configuration files. 

Ensure the 'ds.xml' file uses a connection string and credentials corresponding to the setting for the database driver above

Ensure the '.json' properties file has been updated for the specific instance including:
- content directory
- smtp settings
- context url
- maps and other api keys
- environment identifier

Finally, ensure that the user credential that will run the wildfly service has read/write permissions to the wildfly folder and the content folder created above.

#### Deploying a new version of your Skyve application
Once a Skyve application has been successfully deployed, to update your application with a new version:
- undeploy the previous version or stop the wildfly service
- replace the application '.war' folder with the new version (remove the old version, copy in the new)
- deploy the new version or start the wildfly service

If the server has multiple Skyve application deployments, you can replace one of these without impacting on other deployments by signalling an undeployment and deployment as follows:

To undeploy, create an '.undeploy' file in the wildfly/standalone/deployment/ folder corresponding to the name of your application (an empty text file with that name is all that is required), for example 'helloworld.undeploy'. After approximately 30s, wildlfly will replace this file with a file named 'helloworld.undeployed'. 

To redeploy, create a '.dodeploy' file in the wildfly/standalone/deployment/ folder corresponding to the name of your application, for example 'helloworld.dodeploy' (an empty text file with that name is all that is required). After approximately 30s, wildfly will replace this file with 'helloworld.isdeploying' and once deployment is successful, wildfly will replace this with 'helloworld.deployed' (if successful) or 'helloworld.failed' (if unsuccesful).

### Manually seeding new customer users

#### Creating the customer scope

Here's how to create another customer to the Skyve demo.zip download example, for example a customer called _acme_:
1.    Copy the _demo_ customer folder from ```\demo\skyve\src\skyve\customers\demo``` to ```\demo\skyve\src\skyve\customers\acme```
2.    Rename the file ```\demo\skyve\src\skyve\customers\acme\demo.xml``` to ```\demo\skyve\src\skyve\customers\acme\acme.xml```
3.    Edit the new file ```\demo\skyve\src\skyve\customers\acme\acme.xml``` and change the customer name value to match - i.e. on line 2 ```<customer name="acme" ... ```

All customisations specific to this customer will live within this customer folder.

#### Deploying the customer

If you're not using a development tool (an IDE like eclipse) and you just want another customer in the demo, then you'll need to copy the customer folder manually to the deployment area, i.e. copy ```\demo\skyve\src\skyve\customers\acme``` to  ```\demo\skyve\javaee\skyve.ear\apps.jar\skyve\customers\acme```

If you're using an IDE like eclipse then this copy will happen automatically.

#### Adding a setup user

However to start creating data within that customer context, you'll need a user credential in the database so that you can log in the first time. There's several ways to do this but the simplest at the moment would be to open up the database and run a script - this will inject a user credential. This is just to get you started. Once you've logged in as this user, you can begin adding more users and data via the user interface.

Now start up the application server, by running _\demo\run.bat_ (or \demo\run.sh if you're not using Windows)

##### Using the h2 database manager

If you're using the demo h2 database, to access the h2 database manager embedded in the demo, browse to ```http://localhost:8080/skyve/h2```

![Signing in](/assets/images/h2_database_manager_sign_in.png "Signing in to the embedded h2 database manager")

Check that the JDBC URL matches the name of the h2 file in the ```\demo\content\``` folder - for the demo, this will be _demo_ - and so the JDBC URL should have the full path, for example  ```jdbc:h2:C:/demo/skyve/content/demo``` (where ```C:/demo/skyve/content/demo``` is the full path to the file _demo.h2.db_ - without the ".h2.db")

##### Inserting the user record

NOTE: You will only ever need this script once - to get started - from that point on you won't need to use SQL again to build, test or maintain your application.

Paste in the following lines into your database manager (modify by replacing _acme_ with your customer name):

 ```
 INSERT INTO adm_contact (bizId,bizVersion,bizLock,bizCustomer,bizUserId,bizKey,name,mobile,email1,contactType) VALUES 
 ('setupContact',72,'20080114123937714setup','acme','setup','Setup User','Setup User',NULL,'info@bizhub.com.au','Person');

INSERT INTO adm_securitygroup (bizId,bizVersion,bizLock,bizCustomer,bizUserId,bizKey, name,description) VALUES 
 ('setupGroup',200,'20080114123937714setup','acme','setup','Setup','Setup','Setup user');

INSERT INTO adm_securitygrouprole (bizId,bizVersion,bizLock,bizCustomer,bizUserId,bizKey, roleName,parent_id) VALUES 
 ('setupGroupRoleBasic',69,'20080114123937714setup','acme','setup','admin.BasicUser','admin.BasicUser','setupGroup'),
 ('setupGroupRoleContact',69,'20080114123937714setup','acme','setup','admin.ContactViewer','admin.ContactViewer','setupGroup'),
 ('setupGroupRoleSecurity',69,'20080114123937714demo','acme','setup','admin.SecurityAdministrator','admin.SecurityAdministrator','setupGroup');

INSERT INTO adm_securityuser (bizId,bizVersion,bizLock,bizCustomer,bizUserId,bizKey, userName,password,contact_id) VALUES 
 ('setup',57,'20080114123937698setup','acme','setup','setup','setup','0RGzjA5zvIZ8S61AI2BqDg32TC8=','setupContact');

INSERT INTO adm_securityuser_groups (owner_id,element_id) VALUES  ('setup','setupGroup');
 ```
 
 ![Using h2](/assets/images/h2_database_manager.png "Using the h2 database manager")
 
NOTE: This script assumes the SHA1 password hashing algorithm. If you're using something else, you'll need to swap the value _'0RGzjA5zvIZ8S61AI2BqDg32TC8='_ for the encrypted value using that algorithm.

The above script can be trivially modified if you're using an RDBMS other than h2.

Run the script, then browse to the application at ```localhost:8080\skyve```

Now you can log in as the setup user in the new customer _acme_ - with the following credentials:

- Customer _acme_
- Username _setup_
- Password _password01_

![Customer sign in](/assets/images/skyve_customer_sign_in.png "Skyve customer sign in")

**[⬆ back to top](#contents)**
