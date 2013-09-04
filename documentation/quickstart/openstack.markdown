---
layout: jclouds
title: Quick Start - OpenStack
---

# Quick Start: OpenStack
1. [Introduction](#intro)
1. [Get OpenStack](#openstack)
1. [Get jclouds](#install)
1. [Terminology](#terminology)
1. [List Servers](#nova)
1. [List Containers](#swift)
1. [Next Steps](#next)
1. [OpenStack Providers](#providers)

## <a id="intro"></a>Introduction
[OpenStack](http://www.openstack.org/) is a global collaboration of developers and cloud computing technologists producing the ubiquitous open source cloud computing platform for public and private clouds. The project aims to deliver solutions for all types of clouds by being simple to implement, massively scalable, and feature rich. The technology consists of a series of interrelated projects delivering various components for a cloud infrastructure solution.

## <a id="openstack"></a>Get OpenStack
You can either install a private OpenStack cloud for yourself or use an existing OpenStack public cloud. 

### <a id="devstack"></a>DevStack
If you haven't installed OpenStack before, the best place to start is [DevStack](http://devstack.org/). DevStack is a documented shell script to build complete OpenStack development environments. The 1-2-3 Quick Start doesn't mention it but it's best to create a [localrc](http://devstack.org/samples/localrc.html) file in your devstack directory before running stack.sh. For our purposes the setting we'll need to know to work with jclouds is ADMIN_PASSWORD.

DevStack can be run either in a local VM or in the cloud. The DevStack website has some generic instructions for [Running a Cloud in a VM](http://devstack.org/guides/single-vm.html). You can find more specific instructions to run DevStack in VirtualBox at [Contributing OpenStack Support to jclouds](http://blog.phymata.com/2012/10/03/contributing-openstack-support-to-jclouds/). To run it in the Rackspace Cloud please read [OpenStack devstack on the Rackspace Cloud](http://blog.phymata.com/2012/11/08/openstack-devstack-on-the-rackspace-open-cloud/).

### <a id="clouds"></a>Public Clouds
Because the OpenStack API is also open, the jclouds APIs that talk to private OpenStack clouds work just as well with public OpenStack clouds. OpenStack is used by several large public clouds, both the [HP Cloud](https://www.hpcloud.com/) and [Rackspace Cloud](http://www.rackspace.com/cloud/) are based on it. If you don't want to sign up for a paid public cloud, you can use [TryStack](http://trystack.org/).

## <a id="install"></a>Get jclouds

1. Ensure you are using the [Java Development Kit (JDK) version 6 or later](http://www.oracle.com/technetwork/java/javase/downloads/index.html).
1. In your Terminal or Command Prompt run the following to verify the JDK is installed correctly.
    * `javac -version` 
1. Create a directory to try out jclouds.
    * `mkdir jclouds` 
1. Follow the instructions for [Getting the binaries using Apache Ant](/documentation/userguide/installation-guide#ant).
1. You should now have a directory with the following structure:
    * `jclouds/`
        * `build.xml`
        * `maven-ant-tasks.jar`
        * `lib/`
            * `*.jar`

## <a id="terminology"></a>Terminology
There are some differences in terminology between jclouds and OpenStack that should be made clear.

| jclouds | OpenStack |
|---------|---------------------|
| Compute | Nova
| Node | Server
| Location/Zone | Region
| Hardware | Flavor
| NodeMetadata | Server details
| UserMetadata | Metadata
| BlobStore | Swift
| Blob | File

## <a id="nova"></a>List Servers
### <a id="nova-intro"></a>Introduction

[OpenStack Compute](http://www.openstack.org/software/openstack-compute/) (aka Nova) is an easy to use service that provides on-demand servers that you can use to to build dynamic websites, deliver mobile apps, or crunch big data.

### <a id="nova-source"></a>The Source Code

1. Create a Java source file called JCloudsNova.java in the jclouds directory above.
1. You should now have a directory with the following structure:
    * `jclouds/`
        * `JCloudsNova.java`
        * `build.xml`
        * `maven-ant-tasks.jar`
        * `lib/`
            * `*.jar`
1. Open JCloudsNova.java for editing, read the code below, and copy it in.

<pre>
{% highlight java %}
import static com.google.common.io.Closeables.closeQuietly;
    
import java.io.Closeable;
import java.util.Set;

import org.jclouds.ContextBuilder;
import org.jclouds.compute.ComputeService;
import org.jclouds.compute.ComputeServiceContext;
import org.jclouds.logging.slf4j.config.SLF4JLoggingModule;
import org.jclouds.openstack.nova.v2_0.NovaApi;
import org.jclouds.openstack.nova.v2_0.NovaAsyncApi;
import org.jclouds.openstack.nova.v2_0.domain.Server;
import org.jclouds.openstack.nova.v2_0.features.ServerApi;
import org.jclouds.rest.RestContext;

import com.google.common.collect.ImmutableSet;
import com.google.inject.Module;

public class JCloudsNova implements Closeable {
   private ComputeService compute;
   private RestContext<NovaApi, NovaAsyncApi> nova;
   private Set<String> zones;

   public static void main(String[] args) {
      JCloudsNova jCloudsNova = new JCloudsNova();

      try {
         jCloudsNova.init();
         jCloudsNova.listServers();
         jCloudsNova.close();
      }
      catch (Exception e) {
         e.printStackTrace();
      }
      finally {
         jCloudsNova.close();
      }
   }

   private void init() {
      Iterable<Module> modules = ImmutableSet.<Module> of(new SLF4JLoggingModule());

      String provider = "openstack-nova";
      String identity = "demo:demo"; // tenantName:userName
      String password = "devstack"; // demo account uses ADMIN_PASSWORD too

      ComputeServiceContext context = ContextBuilder.newBuilder(provider)
            .endpoint("http://172.16.0.1:5000/v2.0/")
            .credentials(identity, password)
            .modules(modules)
            .buildView(ComputeServiceContext.class);
      compute = context.getComputeService();
      nova = context.unwrap();
      zones = nova.getApi().getConfiguredZones();
   }

   private void listServers() {
      for (String zone: zones) {
         ServerApi serverApi = nova.getApi().getServerApiForZone(zone);

         System.out.println("Servers in " + zone);

         for (Server server: serverApi.listInDetail().concat()) {
            System.out.println("  " + server);
         }
      }
   }

   public void close() {
      closeQuietly(compute.getContext());
   }
}
{% endhighlight %}
</pre>

In the init() method note that

* `String provider = "openstack-nova";`
  * This ones pretty self explanatory, we're using the OpenStack Nova provider in jclouds
* `String identity = "demo:demo"; // tenantName:userName`
  * Here we use the tenant name and user name with a colon between them instead of just a user name
* `String password = "devstack";`
  *  The demo account uses ADMIN_PASSWORD too
* `.endpoint("http://172.16.0.1:5000/v2.0/")`
  * This is the Keystone endpoint that jclouds needs to connect with to get more info (services and endpoints) from OpenStack
  * When the devstack installation completes successfully, one of the last few lines will read something like "`Keystone is serving at http://172.16.0.1:5000/v2.0/`"
  * Set the endpoint to this URL depending on the method used to get OpenStack above. 

### <a id="nova-compile"></a>Compile and Run

    javac -classpath ".:lib/*" JCloudsNova.java
    
    java -classpath ".:lib/*" JCloudsNova
    
    [logging output]
    Servers in RegionOne
    [logging output]
      Server{uuid=...}
      ...

## <a id="swift"></a>List Containers
### <a id="swift-intro"></a>Introduction

[OpenStack Object Storage](http://www.openstack.org/software/openstack-storage/) (aka Swift) provides redundant, scalable object storage using clusters of standardized servers capable of storing petabytes of data.

### <a id="swift-source"></a>The Source Code

1. Create a Java source file called JCloudsSwift.java in the jclouds directory above.
1. You should now have a directory with the following structure:
    * `jclouds/`
        * `JCloudsSwift.java`
        * `build.xml`
        * `maven-ant-tasks.jar`
        * `lib/`
            * `*.jar`
1. Open JCloudsSwift.java for editing, read the code below, and copy it in.

<pre>
{% highlight java %}
import static com.google.common.io.Closeables.closeQuietly;

import java.io.Closeable;
import java.util.Set;

import org.jclouds.ContextBuilder;
import org.jclouds.blobstore.BlobStore;
import org.jclouds.blobstore.BlobStoreContext;
import org.jclouds.logging.slf4j.config.SLF4JLoggingModule;
import org.jclouds.openstack.swift.CommonSwiftAsyncClient;
import org.jclouds.openstack.swift.CommonSwiftClient;
import org.jclouds.openstack.swift.domain.ContainerMetadata;
import org.jclouds.rest.RestContext;

import com.google.common.collect.ImmutableSet;
import com.google.inject.Module;

public class JCloudsSwift implements Closeable {
   private BlobStore storage;
   private RestContext<CommonSwiftClient, CommonSwiftAsyncClient> swift;

   public static void main(String[] args) {
      JCloudsSwift jCloudsSwift = new JCloudsSwift();

      try {
         jCloudsSwift.init();
         jCloudsSwift.listContainers();
         jCloudsSwift.close();
      }
      catch (Exception e) {
         e.printStackTrace();
      }
      finally {
         jCloudsSwift.close();
      }
   }
   
   private void init() {
      Iterable<Module> modules = ImmutableSet.<Module> of(
            new SLF4JLoggingModule());

      String provider = "swift-keystone";
      String identity = "demo:demo"; // tenantName:userName
      String password = "devstack"; // demo account uses ADMIN_PASSWORD too

      BlobStoreContext context = ContextBuilder.newBuilder(provider)
            .endpoint("http://172.16.0.1:5000/v2.0/")
            .credentials(identity, password)
            .modules(modules)
            .buildView(BlobStoreContext.class);
      storage = context.getBlobStore();
      swift = context.unwrap();
   }

   private void listContainers() {
      System.out.println("List Containers");
      Set<ContainerMetadata> containers = swift.getApi().listContainers();

      for (ContainerMetadata container: containers) {
         System.out.println("  " + container);
      }
   }

   public void close() {
      closeQuietly(storage.getContext());
   }
}
{% endhighlight %}
</pre>

### <a id="swift-compile"></a>Compile and Run

    javac -classpath ".:lib/*" JCloudsSwift.java
    
    java -classpath ".:lib/*" JCloudsSwift
    
    [logging output]
    List Containers
    [logging output]
      ContainerMetadata{name=...}
      ...

## <a id="next"></a>Next Steps


- Try the example above with a keystone endpoint that defines multiple storage regions. For this init() becomes: 
<pre>
{% highlight java %}
private void init() {
   Iterable<Module> modules = ImmutableSet.<Module> of(new SLF4JLoggingModule());

   String provider = "swift-keystone"; //Region selection is limited to swift-keystone provider
   String identity = "demo:demo"; 
   String password = "devstack"; 
   String endpoint = "http://keystone-endpoint.example.com/v2.0"; 
   String region = "RegionOne"; 

   // If your keystone endpoint has multiple storage regions
   // then it is recommended that you specify which region to use. 
   // You can do this via the "jclouds.region" property
   Properties properties = new Properties();
   properties.setProperty(LocationConstants.PROPERTY_REGION, region);

   BlobStoreContext context = ContextBuilder.newBuilder(provider)
         .endpoint(endpoint)
         .credentials(identity, password)
         .modules(modules)
         .overrides(properties)
         .buildView(BlobStoreContext.class);
   storage = context.getBlobStore();
   swift = context.unwrap();
}
{% endhighlight %}
</pre>

- Try the example above with one of the public clouds. For the Rackspace Cloud init() becomes:
<pre>
{% highlight java %}
private void init() {    
   Iterable<Module> modules = ImmutableSet.<Module> of(new SLF4JLoggingModule());
   Properties overrides = new Properties();
   overrides.setProperty(KeystoneProperties.CREDENTIAL_TYPE, CredentialTypes.PASSWORD_CREDENTIALS);
   overrides.setProperty(Constants.PROPERTY_API_VERSION, "2");

   String provider = "openstack-nova";
   String identity = "myUsername"; // userName
   String password = "myPassword"; 

   ComputeServiceContext context = ContextBuilder.newBuilder(provider)
         .endpoint("https://identity.api.rackspacecloud.com/v2.0/")
         .credentials(identity, password)
         .modules(modules)
         .overrides(overrides)
         .buildView(ComputeServiceContext.class);
   compute = context.getComputeService();
   nova = context.unwrap();
   zones = nova.getApi().getConfiguredZones();
}
{% endhighlight %}
</pre>
- Try using the `"openstack-cinder"` provider to list volumes (hint: see [VolumeAndSnapshotApiLiveTest.testListVolumes()](https://github.com/jclouds/jclouds/blob/master/apis/openstack-cinder/src/test/java/org/jclouds/openstack/cinder/v1/features/VolumeAndSnapshotApiLiveTest.java)).
- Have a look at how the optional extensions are handled (hint: see [FloatingIPApiLiveTest.testListFloatingIPs()](https://github.com/jclouds/jclouds/blob/master/apis/openstack-nova/src/test/java/org/jclouds/openstack/nova/v2_0/extensions/FloatingIPApiLiveTest.java#L42)).
- Change the example to do different things that you want to do.
- Browse the [documentation](http://www.jclouds.org/documentation/) and have a look at the [Javadoc](http://demobox.github.com/jclouds-maven-site/latest/apidocs).
- Return to the [Installation Guide](http://www.jclouds.org/documentation/userguide/installation-guide/) and have a look at the different ways to integrate jclouds with your project.
- Join the [jclouds community](http://incubator.apache.org/projects/jclouds.html) as either a developer or user. 


## <a id="providers"></a>OpenStack Providers

This is a list of providers that work with OpenStack that you can use to build your Context.

* `"openstack-nova"`
* `"openstack-keystone"`
* `"openstack-cinder"`
* `"swift-keystone"`
