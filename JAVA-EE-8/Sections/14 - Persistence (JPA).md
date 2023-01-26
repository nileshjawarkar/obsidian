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

## Mappings

### Mapping types

JPA out off the box support following java types and will be automatically mapped to corresponding DB type :
- primitive types such as int, long etc. 
- java.lang.String, Serialisable type including wrappers of simple java types
- Enum types
- java.math.BigInteger, java.math.BigDecimal, java.util.Date, java.util.Calender, java.sql.Date, java.sql.Time, java.sql.Timestamp
- byte[], Byte[], char[], Character[]
- java.time.LocalDate, java.time.LocalTime, java.time.LocalDateTime, java.time.OffsetTime, java.time.OffsetDateTime

### Mapping enum type

By default, Constant's in enum type are assigned a ordinal value. 
Consider example of Color enum. Each color constant in Color enum will have ordinal starting from 0. Initially we have following color's. May be latter, we decided added some more color's  and added them at start or somewhere in middle. 

This will result in change in ordinal values for each constant. If added BLACK before RED, ordinal value for BLACK will be 0 and ordinal value for RED will change to 1. This will lead to inconsistency, if we already have values in database.

``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

public enum Color {
	RED,    //-> 0
	WHITE,  //-> 1
	BLUE,   //-> 2
	ANY     //-> 3
}
```

To avoid this _@Enumerated(EnumType.STRING)_ annotation can be used to store the values in DB as string not as ordinals.

### Mapping large object (Eg. image)

Database can store large objects such images, files etc. _@Lob_ annotation can be used to map those fields, intended to hold large objects. _@Lob_ signifies that the annotated field should be represented as BLOB (binary data) in the DB. JPA supports this annotation on many type, such as String[], char[], byte[], Byte[].

Example - Car having  a picture.
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

...
import javax.persistence.Lob;
...

@Entity
@Table(name = "cars")
public class Car extends BaseEntity {
	...
	@Lob
	private byte[] picture;
	...
}

```

### Transient

While persisting the object to the DB, we sometimes want to ignore certain fields. JPA annotation _@Transient_ can be added to such fields to ignore them.

## Access types

In JPA, mapping information of an entity must be accessible to the ORM provider at runtime, so that when it is writing the data to storage, mapping information can be obtained from the entity instance. Similarly, when the entity state is loaded from the storage, the provider runtime must be able to map the data into a new entity instance.

There are generally two ways to provide this information to ORM providers i.e. `@Access(AccessType.FIELD)` and `@Access(AccessType.PROPERTY)`. But we can use mixed mode as well in some cases.

### Field Access

- To declare field access mode, explicitly annotate the entity with `"@Access(AccessType.FIELD)`“.
- Class fields are used to map the state
- When we use @Id annotation on class field, JPA infer it as a field access mode.
- Please note that getter and setter methods are ignored by the provider.
- All fields must be declared as either _protected_, _package_, or _private_. Public fields are disallowed

### Property Access

- To declare property access mode, explicitly annotate the entity with `"@Access(AccessType.PROPERTY)`“.
-  When property access mode is used, there must be getter and setter methods for the persistent properties.
- The type of property is determined by the return type of the getter method and must be the same as the type of the single parameter passed into the setter method.
- Both methods must be either public or protected visibility. 
- The mapping annotations for a property must be on the getter method.

### Mixed Access

- Though you will not require to use mix modes in most of the scenarios, it is possible and useful in some cases. For example, when an entity subclass is added to an existing hierarchy that uses a different access type.
- Adding an `@Access` annotation with a specified access mode on the subclass entity (or even field) will cause the default access type to be overridden for that entity subclass.

Example -
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

import javax.persistence.Access;

@Entity
@Table(name = "cars")
@Access(AccessType.FIELD) //-- To explicitly define the access
public class Car extends BaseEntity {
	...
}

```