---
permalink: /tutorials/karaf-tutorial/day1/
layout: page
categories: tutorial
tags: [karaf, OSGi, java]
title: Karaf Tutorial Day 1
---

# Hello World!

The tradition commands that all tutorials must have a *Hello World* sample. So let's go and define a very simple sample. This sample will help you to understand the following concepts:

- how to create a bundle
- use OSGi Service-Consumer paradigm
- dependency injection ([Apache Aries Blueprint](http://aries.apache.org/modules/blueprint.html) )
- dynamic configuration
- deploy bundles in Karaf
- OSGi bundle life cycle

### Table of Contents
{:.no_toc}

* Table of Contents Placeholder
{:toc max_level=2}


## Karaf Development setup

I use Maven, I'm always thinking about testing Gradle but I haven't been able to find the time or the project to get started with it.


### Automating bundle manifests with `maven-bundle-plugin`

I usually use the following configuration in my parent pom in order to build my OSGi bundles:

{% highlight xml linenos %}
<plugin>
    <groupId>org.apache.felix</groupId>
    <artifactId>maven-bundle-plugin</artifactId>
    <version>${maven.bundle.plugin.version}</version>
    <extensions>true</extensions>
    <configuration>
        <instructions>
            <Bundle-SymbolicName>${project.artifactId}</Bundle-SymbolicName>
            <Bundle-Version>${project.version}</Bundle-Version>
            <Export-Package>!${bundle.namespace}.internal.*,${bundle.namespace}.*;version="${project.version}"</Export-Package>
            <Private-Package>${bundle.namespace}.internal.*</Private-Package>
            <_include>-osgi.bnd</_include>
        </instructions>
    </configuration>
    <executions>
        <execution>
            <id>bundle-manifest</id>
            <phase>process-classes</phase>
            <goals>
                <goal>manifest</goal>
            </goals>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

This is a generic configuration which simplifies the Manifest creation of OSGi bundles.

As `Bundle-SymbolicName` I use the maven project `artifactId` property and for the `Bundle-Version` I use the maven project `version`.
I set a property `bundle.namespace` with the base package of my bundle classes. Every package under `bundle.namespace` is exported but subpackage `internal` which is kept private.

Additionally I include the osgi.bnd in case I need to set specifically an OSGi manifest entry (i.e. Fragment-Host, etc...)

*More information about `maven-bundle-plugin`can be found [here](http://svn.apache.org/repos/asf/felix/releases/maven-bundle-plugin-2.5.4/doc/site/index.html)*


### Other Maven plugins

Other plugins I use are:


####  `build-helper-maven-plugin`

I use it to upload additional artifacts (besides the bundle itself) to the Maven Repository. In example: config files.

This is the configuration I use in my parent pom.

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <version>${build.helper.plugin.version}</version>
    <executions>
        <execution>
            <id>attach-artifacts</id>
            <phase>package</phase>
            <goals>
                <goal>attach-artifact</goal>
            </goals>
            <configuration>
                <artifacts>
                    <artifact>
                        <file>target/classes/${bundle.namespace}.cfg</file>
                        <type>cfg</type>
                    </artifact>
                </artifacts>
            </configuration>
        </execution>
    </executions>
</plugin>

```

The limitation of this configuration is that the *.cfg file must be named as the base package of the bundle. On the other hand this helps me to know which cfg file belongs to which bundle.

*More information about `build-helpler-maven-plugin`can be found [here](http://mojo.codehaus.org/build-helper-maven-plugin/)*


## *Hello World* Sample

I will build four bundles:

- `hello-world-service`: simple service definition
- `hello-world-service-basic`: simple `HelloService` implementation
- `hello-world-service-config`: another `HelloService` implmentation with configuration
- `hello-world-service-caller`: a `hello-world-service` consumer


### hello-world-service

This is the service definition. Usually we will use a Java(TM) interface in order to implement service definitions.

**org.jemz.karaf.tutorial.hello.service.IHelloService.java**

```java
public interface IHelloService {
    public String getMessage();
}
```
This service just returns a message string.


### hello-world-service-basic

The most basic implementation of `HelloService` is to return a fixed string as shown below:

**org.jemz.karaf.tutorial.hello.service.basic.internal.HelloServiceBasic**

```java
public class HelloServiceBasic implements IHelloService {
    @Override
    public String getMessage() {
      return "I am a HelloServiceBasic!!";
    }
}
```

#### Blueprint

Just define HelloServiceBasic as a IHelloService implementation:

```xml
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
           xmlns:cm="http://aries.apache.org/blueprint/xmlns/blueprint-cm/v1.1.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <bean   id="hello-service-basic"
            class="org.jemz.karaf.tutorial.hello.service.basic.internal.HelloServiceBasic"
            init-method="startup"
            destroy-method="shutdown">
    </bean>

    <service ref="hello-service-basic" interface="org.jemz.karaf.tutorial.hello.service.IHelloService" />

</blueprint>
```

In the bean definition I have added a `init-method` and a `destroy-method` in order to know if the bundle has started or it has been stopped.


### hello-world-service-config

This implementation will return a message got from configuration file:

**org.jemz.karaf.tutorial.hello.service.config.cfg**

```bash

org.jemz.karaf.tutorial.hello.msg="I am a HelloServiceConfig!!"

```

The implementation is as follows

**org.jemz.karaf.tutorial.hello.service.config.internal.HelloServiceConfig**

```java
public class HelloServiceConfig implements IHelloService {

    private static final String HELLO_SERVICE_MSG_CONFIG = "org.jemz.karaf.tutorial.hello.service.msg";
    private String msg = null;

    /**
     * Blueprint: set configuration properties
     */
    public void setHelloServiceConfiguration(Properties properties) {
      msg = (String) properties.get(HELLO_SERVICE_MSG_CONFIG);
    }

    @Override
    public String getMessage() {
      return "*** " + msg + " ***";
    }
}
```

#### Blueprint

Define as IHelloService implementation.Default value for config message and setup of dynamic configuration.

```xml
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
           xmlns:cm="http://aries.apache.org/blueprint/xmlns/blueprint-cm/v1.1.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <cm:property-placeholder persistent-id="org.jemz.karaf.tutorial.hello.service.config" update-strategy="reload" >
        <cm:default-properties>
            <cm:property name="org.jemz.karaf.tutorial.hello.service.msg" value="Hello World!"/>
        </cm:default-properties>
    </cm:property-placeholder>

    <bean   id="hello-service-config"
            class="org.jemz.karaf.tutorial.hello.service.config.internal.HelloServiceConfig"
            init-method="startup"
            destroy-method="shutdown">
        <property name="helloServiceConfiguration">
            <props>
                <prop key="org.jemz.karaf.tutorial.hello.service.msg" value="${org.jemz.karaf.tutorial.hello.service.msg}"/>
            </props>
        </property>
    </bean>

    <service ref="hello-service-config" interface="org.jemz.karaf.tutorial.hello.service.IHelloService" />
</blueprint>

```
The configuration file should be installed under `${karaf.home}/etc`. Changes on the file are dynamically updated on the OSGi bundle.

### hello-world-service-config-admin

In the previous example the message returned from the service implementation was read from a configuration file from a fixed property: `org.jemz.karaf.tutorial.hello.msg`. The bundle receives the updated property value whenever its value changes in the file.

On one side, the bundle `fileinstall` is in charge of watching for the updates in the configuration file and load them into de `ConfigurationAdmin` Service. The bundle is reloaded when a change is detected because of the `policy-strategy=reload` attribute in the blueprint component.

But, what if your bundle can manage additional properties, which we don't want them to be in the configuration file. Maybe a hidden backdoor to do something we don't want anyone to know.

In this sample, our service, will be able to add a prefix and sufix to the message. The prefix and sufix are set by two new hidden properties:

* `org.jemz.karaf.tutorial.hello.service.msg.sufix`
* `org.jemz.karaf.tutorial.hello.service.msg.prefix`

This implementation will return a message got from configuration file:

**org.jemz.karaf.tutorial.hello.service.config.admin.cfg**

```bash

org.jemz.karaf.tutorial.hello.msg="I am a HelloServiceConfig!!"

```

The implementation is as follows

**org.jemz.karaf.tutorial.hello.service.config.admin.internal.HelloServiceConfigAdmin**

```java
public class HelloServiceConfigAdmin implements IHelloService {

    // Service PID - persistent-id
    private static final String HELLO_SERVICE_CONFIG_ADMIN_PID =  "org.jemz.karaf.tutorial.hello.service.config.admin";

    public static final String HELLO_SERVICE_MSG_CONFIG = "org.jemz.karaf.tutorial.hello.service.msg";
    /*
     Hidden properties
     */
     private static final String HELLO_SERVICE_MSG_CONFIG_SUFIX = "org.jemz.karaf.tutorial.hello.service.msg.sufix";
     private static final String HELLO_SERVICE_MSG_CONFIG_PREFIX = "org.jemz.karaf.tutorial.hello.service.msg.prefix";

     private String msg = null;
     private String suffix = "";
     private String prefix = "";

     /**
      * Blueprint reference: init-method
      */
     public void startup() {
         System.out.println("HELLO SERVICE CONFIG STARTED");
     }

     /**
      * Blueprint reference: destroy-method
      */
     public void shutdown() {
         System.out.println("HELLO SERVICE CONFIG SHUTDOWN");
     }

     /**
      * Blueprint: set configuration properties
      */
     public void setHelloServiceConfiguration(ConfigurationAdmin cfgAdmin) {
         try {
             Configuration config = cfgAdmin.getConfiguration(HELLO_SERVICE_CONFIG_ADMIN_PID);
             Dictionary<String, Object> serviceProps = config.getProperties();
             Enumeration<String> en = serviceProps.keys();
             while (en.hasMoreElements()) {
                 String key = en.nextElement();
                 if(key.equals(HELLO_SERVICE_MSG_CONFIG)) {
                     msg = (String) serviceProps.get(key);
                 } else if (key.equals(HELLO_SERVICE_MSG_CONFIG_SUFFIX)) {
                     suffix = (String) serviceProps.get(key);
                 } else if (key.equals(HELLO_SERVICE_MSG_CONFIG_PREFIX)) {
                     prefix = (String) serviceProps.get(key);
                 } else {
                     System.out.println("UNUSED KEY: " + key + " VAL: " + (String)serviceProps.get(key));
                 }
             }
           } catch(NullPointerException e) {
            // Config file doesn't exist
              msg = "Default Message";
          } catch (IOException e) {
              // Ignore error
          }
     }

     @Override
     public String getMessage() {
         return prefix + msg + suffix;
     }
   }
```

The properties of the file are now retrieved from `ConfigurationAdmin` service directly.


#### Blueprint

Define as IHelloService implementation. The `setHelloServiceConfiguration` method now receives the implementation of the current `ConfigurationAdmin` service.

```xml
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
           xmlns:cm="http://aries.apache.org/blueprint/xmlns/blueprint-cm/v1.1.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        >

    <cm:property-placeholder persistent-id="org.jemz.karaf.tutorial.hello.service.config.admin" update-strategy="reload" />

    <bean   id="hello-service-config-admin"
            class="org.jemz.karaf.tutorial.hello.service.config.admin.internal.HelloServiceConfigAdmin"
            init-method="startup"
            destroy-method="shutdown">
        <property name="helloServiceConfiguration" ref="config-admin"/>
    </bean>

    <service ref="hello-service-config-admin" interface="org.jemz.karaf.tutorial.hello.service.IHelloService" />

    <reference id="config-admin" interface="org.osgi.service.cm.ConfigurationAdmin" availability="mandatory"/>

</blueprint>


```
The configuration file should be installed under `${karaf.home}/etc`. Changes on the file are dynamically updated on the OSGi bundle. Now it is possible to add any property to it and it will automatically upload on the bundle.


### hello-world-service-caller

This bundle will consume the `HelloService`:

**org.jemz.karaf.tutorial.hello.service.caller.internal.HelloServiceCaller**

```java
public class HelloServiceCaller {
   private IHelloService helloService = null;

  /**
    * Blueprint reference: bind-method
    * @param serv
    */
   public void setHelloService(IHelloService serv) {
       helloService = serv;
       System.out.println("SET HELLO SERVICE: " + helloService.getMessage());
   }

   /**
    * Blueprint reference: unbind-method
    * @param serv
    */
   public void unsetHelloService(IHelloService serv) {
       helloService = null;
       System.out.println("UNSET HELLO SERVICE");
   }
}
```

#### Blueprint

This bundle will register a reference to a IHelloService implementation:

```xml
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
           xmlns:cm="http://aries.apache.org/blueprint/xmlns/blueprint-cm/v1.1.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <bean   id="hello-service-caller"
            class="org.jemz.karaf.tutorial.hello.service.caller.internal.HelloServiceCaller"
            init-method="startup"
            destroy-method="shutdown"/>

    <reference id="hello-service" interface="org.jemz.karaf.tutorial.hello.service.IHelloService"
               availability="optional">
        <reference-listener bind-method="setHelloService"
                            unbind-method="unsetHelloService">
            <ref component-id="hello-service-caller"/>
        </reference-listener>
    </reference>
</blueprint>
```

## Installing HelloService sample in Karaf

Basically you can install your bundles in a Karaf installation in multiple different ways.

The simplest one is to place all bundle jar files under ${karaf.home}/deploy folder. OSGi bundles will be automatically installed
and started.

Another way is using the Maven descriptor or using a URL to the jar files.

Here I will use the Maven descriptor as:
  `bundle:install mvn:<groupid>/<artifactId>/<version>[/<type>/<classifier>]`

```bash
karaf@root()> install mvn:org.jemz.karaf.tutorial/hello-world-service/1.0.0-SNAPSHOT
Bundle ID: 64
```

The bundle is installed with ID 64, but it is not yet started:

```bash
karaf@root()> list
START LEVEL 100 , List Threshold: 50
ID | State     | Lvl | Version        | Name
-----------------------------------------------------------
64 | Installed |  80 | 1.0.0.SNAPSHOT | hello-world-service
```

You can start the bundle with command: `bundle:start <bundleID>`. It is also possible to install and start a bundle with
the install command plus `-s` flag.

```bash
karaf@root()> start 64
karaf@root()> list
START LEVEL 100 , List Threshold: 50
ID | State  | Lvl | Version        | Name
--------------------------------------------------------
64 | Active |  80 | 1.0.0.SNAPSHOT | hello-world-service
```

It may be useful to check the manifest of the bundle to see which packages, services it is exporting/importing, or any other
information.

```bash
karaf@root()> headers 64

hello-world-service (64)
------------------------
Created-By = Apache Maven Bundle Plugin
Manifest-Version = 1.0
Bnd-LastModified = 1432134646433
Build-Jdk = 1.8.0_40
Built-By = jmartinez
Tool = Bnd-2.3.0.201405100607

Bundle-ManifestVersion = 2
Bundle-SymbolicName = hello-world-service
Bundle-Version = 1.0.0.SNAPSHOT
Bundle-Name = hello-world-service
Bundle-Description = Karaf Tutorial :: hello-world-service
Bundle-DocURL = https://jomarmar.github.io
Bundle-Vendor = JemZ

Require-Capability =
	osgi.ee;filter:=(&(osgi.ee=JavaSE)(version=1.8))

Export-Package =
	org.jemz.karaf.tutorial.hello.service;version=1.0.0.SNAPSHOT

```

Also you can check if the package is really exported with `exports` command and which bundle is exporting it.

```bash
karaf@root()> exports | grep org.jemz
org.jemz.karaf.tutorial.hello.service          | 1.0.0.SNAPSHOT | 64 | hello-world-service
```

Now it is time to install `hello-world-service-basic` implementation:

```bash
karaf@root()> install -s mvn:org.jemz.karaf.tutorial/hello-world-service-basic/1.0.0-SNAPSHOT
HELLO SERVICE BASIC STARTED
Bundle ID: 65
```

You can check the IHelloService implementation by installing the `hello-world-service-caller` consumer:

```bash
karaf@root()> install -s mvn:org.jemz.karaf.tutorial/hello-world-service-caller/1.0.0-SNAPSHOT
HELLO SERVICE CALLER STARTED
SET HELLO SERVICE: I am a HelloServiceBasic!!
Bundle ID: 66
```

Place `org.jemz.karaf.tutorial.hello.service.config.cfg` in ${karaf.home}/etc folder and install `hello-world-service-config`.
As you can see the new service is not consumed by the caller as it has not been started yet.

```bash
karaf@root()> install mvn:org.jemz.karaf.tutorial/hello-world-service-config/1.0.0-SNAPSHOT
Bundle ID: 67
```

If you stop `hello-world-service-basic` bundle and then start the `hello-world-service-config` one is automatically registered by the `hello-world-service-caller` service.

```bash
karaf@root()> stop 65
UNSET HELLO SERVICE
HELLO SERVICE BASIC SHUTDOWN
karaf@root()> start 67
HELLO SERVICE CONFIG STARTED
SET HELLO SERVICE: *** "I am a HelloServiceConfig!!" ***
karaf@root()>
```
If you edit `org.jemz.karaf.tutorial.hello.service.config.cfg`, the service `hello-world-service-config` is reloaded with the new value:

```bash
karaf@root()> UNSET HELLO SERVICE
HELLO SERVICE CONFIG SHUTDOWN
HELLO SERVICE CONFIG STARTED
SET HELLO SERVICE: *** "I am a HelloServiceConfig EDITTED!!" ***
```
Finally if you start `hello-world-service-basic` it won't be registered by the caller. That is because in the blueprint I have defined the consumer as
a `reference` tag and not as a `reference-list`. You can find more information about blueprint [here](http://aries.apache.org/modules/blueprint.html)
or at this [IBM's article](http://www.ibm.com/developerworks/library/os-osgiblueprint/).
The service is automatically registered whenever the `hello-world-service-config` is stopped.

```bash
karaf@root()> start 65
HELLO SERVICE BASIC STARTED
karaf@root()> stop 67
SET HELLO SERVICE: I am a HelloServiceBasic!!
HELLO SERVICE CONFIG SHUTDOWN
```
### Playing with hello-world-service-config-admin

Installing `hello-world-service-config-admin`

```bash
karaf@root()> install -s mvn:org.jemz.karaf.tutorial/hello-world-service/1.0.0-SNAPSHOT
Bundle ID: 64
karaf@root()> install -s mvn:org.jemz.karaf.tutorial/hello-world-service-config-admin/1.0.0-SNAPSHOT
HELLO SERVICE CONFIG STARTED
Bundle ID: 65
karaf@root()> install -s mvn:org.jemz.karaf.tutorial/hello-world-service-caller/1.0.0-SNAPSHOT
HELLO SERVICE CALLER STARTED
SET HELLO SERVICE: Default Message
Bundle ID: 66
```
`Default Message` is displayed as we haven't install the configuration file yet. Copy from resources `org.jemz.karaf.tutorial.hello.service.config.admin.cfg` to $KARAF_HOME/etc folder.

Now the displayed message is: `"I am a HelloServiceConfigAdmin!!"`. Now you can add more properties to the file like:

```
org.jemz.karaf.tutorial.hello.service.msg="I am a HelloServiceConfigAdmin!!"
org.jemz.karaf.tutorial.hello.service.msg.suffix=***
org.jemz.karaf.tutorial.hello.service.msg.prefix=---
```

Then the displayed message will be: `---"I am a HelloServiceConfigAdmin!!"***`.
It is also possible to add new properties from the console as follows:

```
karaf@root()> config:edit org.jemz.karaf.tutorial.hello.service.config.admin
karaf@root()>        
karaf@root()> config:property-list
   felix.fileinstall.filename = file:/home/jmartinez/temp/apache-karaf-3.0.4/etc/org.jemz.karaf.tutorial.hello.service.config.admin.cfg
   org.jemz.karaf.tutorial.hello.service.msg = "I am a HelloServiceConfigAdmin!!"
   org.jemz.karaf.tutorial.hello.service.msg.prefix = ---
   org.jemz.karaf.tutorial.hello.service.msg.suffix = ***
   service.pid = org.jemz.karaf.tutorial.hello.service.config.admin
karaf@root()> config:property-set testprop testvalue
karaf@root()>        
karaf@root()> config:update
UNSET HELLO SERVICE
HELLO SERVICE CONFIG SHUTDOWN
karaf@root()> UNUSED KEY: felix.fileinstall.filename VAL: file:/home/jmartinez/temp/apache-karaf-3.0.4/etc/org.jemz.karaf.tutorial.hello.service.config.admin.cfg
UNUSED KEY: service.pid VAL: org.jemz.karaf.tutorial.hello.service.config.admin
UNUSED KEY: testprop VAL: testvalue
HELLO SERVICE CONFIG STARTED
SET HELLO SERVICE: ---"I am a HelloServiceConfigAdmin!!"***

karaf@root()>

```

## Userful Karaf console commands

config:edit [PID] - Edit config file for PID

config:property-list - List properties in config file

config:property-set [key] [value] - Sets a new value or new property if it doesn't exist.

config:update - Write changes on the file.


## What's next

You can practice with these bundles to see how blueprint injection works and learn about OSGi's lifecycle. Tomorrow we will start a new project based on
multiple bundles implementing the same service. I will also show you how Karaf console can be easily extended.
