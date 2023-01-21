It is used to persist data between Java object and relational database. JPA acts as a bridge between object-oriented domain models and relational database systems. As JPA is just a specification, it doesn't perform any operation by itself. It requires an implementation. So, ORM tools like Hibernate, TopLink and iBatis implements JPA specifications for data persistence.

### Configuring the DB
TomEE server (_may be other severs also_) by default comes with HSQLDB. In this section, we will make use of that it. Java EE server uses "persistence.xml" file for DB configuration.

persistence.xml - Basic configuration required to use HSQLDB
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

**Note** - For  Maven war projects, keep persistence.xml at "src/main/resources/META-INF/persistence.xml". _After deployment_ in the JavaEE server,  server expect this file at "\<APP\>/WEB-INF/classes/META-INF/persistence.xml". 
**Note** - After setting "exclude-unlisted-classes"  to "false", actually there is no need to list entity classes in persistence.xml, but some how it is not working for tomee, that's why we can see entity classes in the xml.

