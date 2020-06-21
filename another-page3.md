---
layout: default
---


## Maven environment configuration
First you need to [download the Maven][jekyll-docs2], what i'am using is the MacOS, so i input:
```$xslt
# wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
# tar -xvf  apache-maven-3.3.9-bin.tar.gz
# sudo mv -f apache-maven-3.3.9 /usr/local/
```
## Maven POM
POM (Project Object Model, Project Object Model) is the basic unit of work of Maven project, is an XML file, contains the basic information of the project, used to describe how the project is built, declare the project dependencies, and so on.
When executing a task or goal, Maven looks for the POM in the current directory. It reads the POM, obtains the required configuration information, and then executes the target.
The following configurations can be specified in POM:
```
Project dependency
Plugin
Execution goals
Project build profile
Project version
List of project developers
Related mailing list information
```
All POM files require a project element and three required fields: groupId, artifactId, and version. For example:
```$xslt
<project xmlns = "http://maven.apache.org/POM/4.0.0"
    xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>   
    <groupId>com.companyname.project-group</groupId>    
    <artifactId>project</artifactId>
    <version>1.0</version>
</project>
```
Super POM is Maven's default POM. All POMs inherit from a super POM. The super POM contains some default settings that can be inherited. Therefore, when Maven finds that it needs to download the dependencies in the POM, it will go to the default repository http://repo1.maven.org/maven2 configured in Super POM to download.
## Maven build life cycle
The Maven build life cycle defines a project build and release process.
A typical Maven build life cycle is composed of a sequence of the following stages:
```
Validate : project Validate the project is correct and all required information is available
Compile : compilation Source code compilation is completed at this stage
Test : Use the appropriate unit test framework (such as JUnit) to run the test.
Package : create JAR/WAR package as defined in pom.xml
verify : the results of the integration test to ensure that the quality is up to standard
Install : the packaged project to a local warehouse for use by other projects
Deploy : copy the final project package to the remote repository to share with other developers and projects
```
## Maven build configuration file
Maven build configuration file
The build configuration file is a series of configuration item values that can be used to set or override Maven build default values.
Using the build configuration file, you can customize the build method for different environments, such as production environment and development environment.
The configuration file is specified in the pom.xml file using activeProfiles or profiles elements, and can be triggered in various ways. The configuration file modifies the POM during construction and is used to set different target environments for the parameters
## Maven repository
In Maven terminology, a warehouse is a place.
The Maven repository is a third-party library that the project depends on. The location of this library is called the repository.
In Maven, the output of any dependency, plugin, or project build can be called a component.
The Maven repository can help us manage components (mainly JAR), it is where all JAR files (WAR, ZIP, POM, etc.) are placed.
There are three types of Maven repositories:
##### Local
Maven's local repository will not be created after Maven is installed. It is created when the maven command is executed for the first time.
When running Maven, any components needed by Maven are obtained directly from the local repository. If the local warehouse does not have it, it will first try to download the components from the remote warehouse to the local warehouse, and then use the components of the local warehouse.
By default, regardless of Linux or Windows, each user has a repository directory with a path name of .m2/respository/ in their own user directory.
##### Central
The Maven central warehouse is a warehouse provided by the Maven community, which contains a large number of commonly used libraries.
The central warehouse contains most popular open source Java components, as well as source code, author information, SCM, information, license information, etc. In general, the components that a simple Java project depends on can be downloaded here.
The key concepts of the central warehouse:
This warehouse is managed by the Maven community.
No configuration is required.
You need to access it through the network.
To browse the contents of the central warehouse, the maven community provides a [URL][jekyll-docs2]. Using this repository, developers can search all available code bases.
##### Remote
If Maven cannot find the dependent files in the central repository, it will stop the build process and output an error message to the console. To avoid this situation, Maven provides the concept of a remote warehouse, which is a custom warehouse for developers, which contains the required code base or jar files used in other projects.
For example, using the following pom.xml, Maven will download the dependent (not available in the central repository) file declared in the pom.xml from the remote repository.

Note: The Maven warehouse defaults to foreign countries, and domestic use is unavoidably slow. We can change to the Alibaba Cloud warehouse by Modify the setting.xml file in the conf folder under the maven root directory, and add the following on the mirrors node:
```$xslt
<mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
</mirrors>
```
And in Pom.xml:
```$xslt
<repositories>  
        <repository>  
            <id>alimaven</id>  
            <name>aliyun maven</name>  
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
            <releases>  
                <enabled>true</enabled>  
            </releases>  
            <snapshots>  
                <enabled>false</enabled>  
            </snapshots>  
        </repository>  
</repositories>
```

## Maven build Java project
Input:
```$xslt
mvn archetype:generate "-DgroupId=com.companyname.bank" "-DartifactId=consumerBanking" "-DarchetypeArtifactId=maven-archetype-quickstart" "-DinteractiveMode=false"
```
Open the ...\MVN\consumerBanking\src\test\java\com\companyname\bank folder, you can see the Java test file AppTest.java. In the following development process, we only need to place it according to the structure mentioned in the table above, and other things will be done by Maven for us.
## Maven build & project testing
Open the consumerBanking folder. You will see there is a pom.xml file

Execute the following mvn command to start building the project:
```$xslt
mvn clean package
```
After the execution, we have built our own project and created the final jar file

## Maven project template
Input:
```$xslt
mvn archetype:generate 
```
Maven will start processing and ask to select the required prototype and after that, Maven automatically generates a pom.xml file for the project.

The power of Maven is that you can use maven simple commands to create any type of project, and you can start your development.
## Maven project documentation
Maven uses the following command to quickly create a java project:
```$xslt
mvn archetype:generate -DgroupId=com.companyname.bank -DartifactId=consumerBanking -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```
Open the consumerBanking folder and execute the following mvn command:
```$xslt
mvn site
```
Open the ...\MVN\consumerBanking\target\site folder. Click index.html to see the document
## Maven snapshot (SNAPSHOT)
A snapshot is a special version that specifies a copy of a current development progress. Unlike the regular version, Maven checks new snapshots in the remote repository every time it is built. Now the data-service team will release a snapshot of the updated code to the warehouse every time, for example, data-service:1.0-SNAPSHOT to replace the old snapshot jar package.
## Maven automated deployment
During the project development process, the deployment process includes the following steps:
```
1.Submit all project code to SVN or code base and label it.
2.Download the complete source code from SVN.
3.Build the application.
4.Store the WAR or EAR file from the build to a common network location.
5.Obtain files from the network and deploy the files to the production site.
6.Update the document and update the version number of the application.
```
The problem is that Usually the above mentioned development process involves multiple teams. One team may be responsible for submitting code, another team is responsible for building, etc. It is likely that any step may go wrong due to the human operations involved and the multi-team environment. For example, the older version was not updated on the network machine, and the deployment team redeployed the earlier build version. 

Above all, Automate deployment can be done by combining the following solutions:
              Use Maven to build and publish projects
              Use SubVersion, source code repository to manage source code
              Use remote warehouse management software (Jfrog or Nexus) to manage project binaries.

## Maven IntelliJ
It can import a new Maven project in IntellJ IDEA.

Some features of IntelliJ IDEA are listed below:
```
You can run Maven goals through IntelliJ IDEA.
You can view the output of Maven commands in IntelliJ IDEA's own terminal.
You can update Maven's dependencies in the IDE.
You can start the Maven build in IntelliJ IDEA.
IntelliJ IDEA is based on Maven's pom.xml to automate management of dependencies.
IntelliJ IDEA can solve the dependency problem of Maven through its own workspace, without installing to the local Maven repository, although the dependent projects are in the same workspace.
IntelliJ IDEA can automatically download the required dependencies and source code from the remote Maven repository.
IntelliJ IDEA provides a wizard for creating Maven projects and pom.xml files.
```
## Thoughts
So far I have learned the main functions of Maven are : project construction; project dependency management; software project continuous integration; version management; project site description and information management, which gives us a lot of convenience.

It provides an idea for the team to manage and build projects more scientifically. Use the configuration file to describe the project description, name, version number, project dependencies, etc. Make the project description structure clear, and the cost for anyone to take over is relatively low.

[jekyll-docs2]: http://maven.apache.org/download.cgi
[jekyll-docs1]: http://search.maven.org/#browse

[back](./)