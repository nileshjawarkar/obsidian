
1) Create project without using any artifact
2) Modify POM.xml
	- Change packaging to war
	- Add following text to configure java 17 and war plugin.

``` xml
<plugins>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-compiler-plugin</artifactId>
		<version>3.10.1</version>
		<configuration>
		<source>17</source>
		<target>17</target>
		<showWarnings>true</showWarnings>
		<compilerVersion>17</compilerVersion>
		<debug>true</debug>
		</configuration>
	</plugin>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-surefire-plugin</artifactId>
		<version>2.22.2</version>
	</plugin>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-war-plugin</artifactId>
		<version>3.3.1</version>
	</plugin>
</plugins>

```

3) Add JavaEE 8 dependency
``` xml
<dependency>
	<groupId>javax</groupId>
	<artifactId>javaee-api</artifactId>
	<version>8.0</version>
	<scope>provided</scope>
</dependency>
```


4) By default Eclipse/Maven will create following directory structure
```
ProjectRoot
	|
	+-- src
		|
		+-- main
			|
			+-- java
			|
			+-- resources
		|
		+-- test
			|
			+-- java
			|
			+-- resources

```

5) Add webapp directory under src/main. Under webapp directory create WEB-INF directory. Add META-INF directory under "src/main/resources". 
```
ProjectRoot
	|
	+-- src
		|
		+-- main
			|
			+-- java
			|
			+-- resources
				|
				+-- META-INF
					|
					+-- persistence.xml			
			|
			+-- webapp
				|
				+-- WEB-INF
					|
					+-- beans.xml
					|
					+-- web.xml


```

6) Add following contents to beans.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
bean-discovery-mode="all">
</beans>
```

7) Add following content to the persistence.xml. Note persistence.xml must be added to "src/main/resources/META-INF" directory, otherwise we can face wired error like "Can't find a persistence unit named null in deployment".
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
xmlns="http://xmlns.jcp.org/xml/ns/persistence"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
<persistence-unit name="prod" transaction-type="JTA">
<!-- <exclude-unlisted-classes>false</exclude-unlisted-classes> -->
<class>com.nilesh.jawarkar.learn.javaee8.entity.Car</class>
<properties>
<property
name="javax.persistence.schema-generation.database.action"
value="drop-and-create" />
</properties>
</persistence-unit>
</persistence>
``` 

8) Added following content to web.xml. Even if we do not add web.xml, in most cases it will work. But for sake of completeness, I am adding it. 
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE web-app PUBLIC '-//Sun Microsystems, Inc.//DTD Web
Application 2.3//EN' 'http://java.sun.com/dtd/web-app_2_3.dtd'>
<web-app>
	<display-name>carman</display-name>
</web-app>
```

9) Optional - When web.xml is not added please set following property in pom.xml
``` xml
<properties>
	<failOnMissingWebXml>false</failOnMissingWebXml>
</properties>
```
#### Now Java EE project is ready to work on. Happy learning.









