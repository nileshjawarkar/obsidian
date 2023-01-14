
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

5) Add webapp directory along with WEB-INF, META-INF directory under src/main as follows. 
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
			+-- webapp
				|
				+-- WEB-INF
					|
					+-- beans.xml
				+-- META-INF
					|
					+-- persistence.xml

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

7) Add following content to the persistence.xml
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

#### Now Java EE project is ready to work on. Happy learning.









