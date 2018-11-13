ORMLite Eclipse Plug-in
=======================

This project compiles [ORMLite Core](https://github.com/j256/ormlite-core) and [JDBC](https://github.com/j256/ormlite-jdbc) packages into a single Eclipse plug-in 
and creates a P2 repository to publish it as the "Ormlite 5.1" feature. 
The ORMLite Eclipse Plug-in is the key component of the [Classy Data Lightweight JPA](http://cybersearch2.com.au/develop/classydata.html) library bieng applied to 
an Android Development IDE to persist project configurations. Classy Data is a minimal JPA implementation layered over the top of ORMLite. Both Classy Data and ORMLite 
are good candidates for Eclipse modularization as they share the same lightweight, standard-Java-only ethos.

Any application of the ORMLite Eclipse Plug-in needs to adapt to a standard application interface which applies to all plug-ins. You can find out more in 
the Vogella [OSGi Modularity - Tutorial](http://www.vogella.com/tutorials/OSGi/article.html). This restricts access to classes from one plug-in to other plugins
and prevents discovery of classes to be persisted as implemented in ORMLite. Classy Data overcomes this problem by having entity classes declared in a peristence 
XML configuration file and using a class loader provided by it's client to instantiate them.

Another consideration is that the number of database drivers available on [Orbit](http://download.eclipse.org/tools/orbit/downloads/drops/R20180829150157), 
the Eclipse plug-in repository is limited to h2 (org.h2 v1.3.168) and derby (org.apache.derby v10.11.1.1). As h2 is probaly better known, it is included in the 
ORMLite Eclipse Plug-in dependencies and selected as the database to be used by the Classy Data plugin. 

## Usage in Classy Data

Although Classy Data places a JPA application layer on top of ORMLite, it does not hide it because all queries are based on ORMLite's SQL query "compiler",
which provides comprehensive support for programmatically generating SQL queries. Therefore, ORMLite Eclipse Plug-in is automatically a dependency where
ever Classy Data is a dependency. Here is an actual code snippet where an ORMLite QueryBuilder object is modified by appending a where clause to select
an object by primary key named "id":

    // build a query with the WHERE clause set to 'id = ?'
    queryBuilder.where().eq("id", primaryKeyArg);

Another ORMLite exposure is through the EntityManager getDelegate() method which returns a helper object to provide access to ORMLite Database
Access Objects (DAOs). Here is the helper's signature:

    /**
     * Returns ORMLite DAO for specified entity class
     * @param clazz  Entity class
     * @return PersistenceDao
     * @throws UnsupportedOperationException if class is unknown for this persistence unit
     */
    public PersistenceDao<?, ?> getDaoForClass(Class<?> clazz)
    {
    ...
    }
    
This allows customization where the JPA implementation is not sufficient.

## Build Instructions

The build project is configured to work with the latest ORMLite version at time of creation, which is version 5.1. The resulting plugin version is 5.1.0 in accordance with Eclipse conventions.
There are no other build artifacts apart from the P2 repository.

### Prequisites

* Java 8 
* [Maven](https://maven.apache.org/download.cgi) - v3.5.2+ recommended
* [Git](https://git-scm.com/downloads)
* Set up Maven toolchains configuration to point to your Java 8 JDK installation. 
Refer [Guide to Using Toolchains](https://maven.apache.org/guides/mini/guide-using-toolchains.html) and [JDK Toolchain](http://maven.apache.org/plugins/maven-toolchains-plugin/toolchains/jdk.html)

You should also ensure your Git [username](https://help.github.com/articles/setting-your-username-in-git/) and 
[commit email address](https://help.github.com/articles/setting-your-commit-email-address-in-git/) are configured.

### Steps

1. Clone this repository to your workspace and go to the newly created "com.j256.ormlite.eclipse" project sub directory. 
1. From a command terminal, launch a Maven build using parameters you will find in the provided build.sh file. If on Linux, you can run build.sh as a shell command.
1. If a first time build, you will potentially observe a large number of files being downloaded to be cached on the local Maven repository. If so, it may be time for a coffee break.

The build, when successful, creates a P2 repository in project sub directory releng/ormlite.update/target/repository.


### Build Project Details

The build project is created guided by the Vogella [Eclipse Tycho for building plug-ins, OSGi bundles and Eclipse applications - Tutorial](http://www.vogella.com/tutorials/EclipseTycho/article.html).
Tycho is a Maven plug-in library specifically for building Eclipse products. It has a highly engineered approach which requires a lot of project structure, even for a modest plug-in.

The project structure is:

* / (root) Top- level POM declares group id, artifact id, version and some project information.  Note the SNAPSHOT version convention is always used, even for releases.
* /bundle/ormlite.plugin The actual plug-in project. Note the sources are not included in the distribution, but instead are downloaded and unpacked from the Maven Central repository.
* /feature/com.j256.feature The feature definition which provides information needed to install the plug-in, including the license text.
* /releng/ormlite.configuration Configures Tycho, Toolchains and software versions
* /releng/ormlite.target Contains the target configuration which identifies which features are to be made available for building the plugin. Ensures build consistency over time.
* /releng/ormlite.update The sub project which builds the P2 repository. The plug-in artifacts are given a unique identity, base on a timestamp, for each build.

As the sources are downloaded from the Maven Central repository, it ensures that a release version of ORMLite is being built, and not a snapshot, as happens when sources come from GitHub.
 
Note that it has also been considered to build separate Core and Jbdc plug-ins derived from the jar counterparts, but this proved unworkable because the two jars share the same packages.
Eclipse does not allow a Java package to be split across 2 plug-ins because it breaks modularity.

### Trouble Shooting

If the expected "Build successful" message does not appear at the end of the build, you must try to understand the error information provided by Maven. Generally, things to check:

1. All required command parameters correctly entered (refer build.sh file in root location).
1. The machine has internet access, there are no internet outages and, if required, the http proxy are details configured in Maven settings file.
1. Check the toolchains.xml file, co-located with the Maven settings file, is correctly configured. Note that all JDK Toolchain items must be correct even though only one is used at any time.
1. If the Java compile build stage fails, delete the contents of directories /bundle/ormlite.plugin/src and /bundle/ormlite.plugin/libs and start the build again. This checks for a file corrupted on download/unpack.

