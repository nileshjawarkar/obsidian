- It is used to persist data between Java object and relational database. 
- JPA acts as a bridge between object-oriented domain models and relational database systems. 
- JPA is just a JAVA EE specification, it doesn't perform any operation by itself. 
- It requires ORM tools like Hibernate, TopLink and iBatis which implements JPA specifications for data persistence.

### ORM
- Object Relational Mapping (ORM) is a functionality which is used to develop and maintain a relationship between an object and relational database by mapping an object state to database column. 
- It is capable to handle various database operations easily such as inserting, updating, deleting etc.

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


## JPA entity

Entity is an application-defined object and it must have following properties :
	 **Persistability** - It is stored in the database and can be accessed anytime
	 **Unique Identity** - When the object is stored in the database, it must have unique identity. This unique object identity is equivalent to primary key in database.
	 **Transactionality** - Entity can perform various operations such as create, delete, update. Each operation makes some changes in the database. It ensures that whatever changes made in the database either be succeed or failed atomically.

A Java class can be easily transformed into an entity. For transformation the basic requirements are:
	1) No-argument Constructor
	2) @Entity annotation
	3) @Id annotation to define unique identity

``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.Id;
import javax.persistence.NamedQueries;
import javax.persistence.NamedQuery;
import javax.persistence.Table;

@Entity
@Table(name = "cars")
public class Car {
	@Id
	private String             id;

	@Enumerated(EnumType.STRING)
	private EngineType         engineType;
	@Enumerated(EnumType.STRING)
	private Color              color;

	public Color getColor() {
		return color;
	}

	public EngineType getEngineType() {
		return engineType;
	}

	public String getId() {
		return id;
	}

	public void setColor(final Color color) {
		this.color = color;
	}

	public void setEngine(final EngineType engineType) {
		this.engineType = engineType;
	}

	public void setEngineType(final EngineType engineType) {
		this.engineType = engineType;
	}

	public void setId(final String id) {
		this.id = id;
	}

	public void setIdentifier(final String id) {
		this.id = id;
	}
}
```

### Customising table mapping

Optionally, we can define table name to be used for entity using @Table annotation. By default it will be entities name (class name).

### Using super class

If all the entities has some common properties (such as id), we can create super class with these common properties. Lets define a super class for out Car class and modify Car class to use it as a super class :

``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

import javax.persistence.Id;
import javax.persistence.MappedSuperclass;

@MappedSuperclass
public class BaseEntity {
	@Id
	protected String id;
	
	public String getId() {
		return this.id;
	}
	
	public void setId(final String id) {
		this.id = id;
	}
}
```

Modified Car class :
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.NamedQueries;
import javax.persistence.NamedQuery;
import javax.persistence.Table;

@Entity
@Table(name = "cars")
@AttributeOverride(name = "id", column = @Column(name = "carId"))
public class Car extends BaseEntity {
	@Enumerated(EnumType.STRING)
	private EngineType engineType;
	@Enumerated(EnumType.STRING)
	private Color      color;

	public Color getColor() {
		return this.color;
	}

	public EngineType getEngineType() {
		return this.engineType;
	}

	public void setColor(final Color color) {
		this.color = color;
	}

	public void setEngine(final EngineType engineType) {
		this.engineType = engineType;
	}

	public void setEngineType(final EngineType engineType) {
		this.engineType = engineType;
	}
}

```

### Overriding super class properties

Annotation @AttributeOverride can be used to override attribute names. In example shown above, we overridden id by carId. When data stored in DB, it will be stored in carId column.


