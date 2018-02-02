Title: Maven: a lot better then Ant
Date: 2008-04-04 22:53
Author: Surya
Category: Rant
Slug: maven-a-lot-better-then-ant
Status: published

Maven is a software tool for Java project management and build
automation similar in functionality to the Apache Ant. Now a days a lot
of open source java project are using Maven.  
Some of the main features are:-

  - Making the build process easy and it is network-ready
  - A way to share JARs across several projects
  - Providing guidelines for best practices development i.e. to write
    unit test
  - Creates common configuration for eclipse for users of same project.
  - [more](http://maven.apache.org/maven-features.html)...

To get started go here
<http://maven.apache.org/guides/getting-started/index.html>  
also checkout <http://en.wikipedia.org/wiki/Apache_Maven>

A sample web project would look like  
/project-name  
/project-name/src/main/java  
/project-name/src/main/resources  
/project-name/src/test/java  
/project-name/src/test/resources  
/project-name/pom.xml  
/project-name/src/main/webapp/project-name/src/main/webapp/WEB-INF

Some of the commands i use

mvn eclipse:eclipse  
mvn clean  
mvn install  
[more](http://maven.apache.org/plugins/index.html) ...
