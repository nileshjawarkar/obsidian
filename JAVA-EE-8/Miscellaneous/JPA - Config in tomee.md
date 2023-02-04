
1) **Define resource - in `WEB-INF/resources.xml`**
- Example 01 - Oracle
``` xml
<tomee>
<Resource id="jdbc/carman" type="DataSource">
    JdbcDriver  oracle.jdbc.OracleDriver
    JdbcUrl jdbc:oracle:thin:@localhost:1521:carman
    UserName    carman
    Password    carman123
</Resource>
</tomee>
```

- Example 02 - Derby Embedded Driver
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<tomee>
<Resource id="jdbc/carman" type="DataSource">
    JdbcDriver org.apache.derby.jdbc.EmbeddedDriver
    JdbcUrl jdbc:derby:carman;create=true
    UserName carman
    Password carman123
</Resource>
</tomee>
```

- Example 03 - Derby Network Driver
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<tomee>
<Resource id="jdbc/carman_managed" type="DataSource">
    JdbcDriver org.apache.derby.jdbc.ClientDriver
    JdbcUrl jdbc:derby://localhost:1527/cardb;create=true
    UserName carman
    Password carman123
</Resource>
<Resource id="jdbc/carman_unmanaged" type="DataSource">
    JdbcDriver org.apache.derby.jdbc.ClientDriver
    JdbcUrl jdbc:derby://localhost:1527/cardb;create=true
    UserName carman
    Password carman123
    JtaManaged false
</Resource>
</tomee>
```

2) **Use resource in - `META-INF/persistence.xml`**
- Example 01
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
	xmlns="http://xmlns.jcp.org/xml/ns/persistence"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
	<persistence-unit name="prod" transaction-type="JTA">
	    <jta-data-source>jdbc/carman_managed</jta-data-source>
	    <non-jta-data-source>jdbc/carman_unmanaged</non-jta-data-source>
		<exclude-unlisted-classes>false</exclude-unlisted-classes>
		<properties>
			<property 
			     name="javax.persistence.schema-generation.database.action"
			     value="drop-and-create" />
			</properties>		
	</persistence-unit>
</persistence>
```


## Critical: Always set jta-data-source and non-jta-data-source

- Always set the value of jta-data-source and non-jta-data-source in your persistence.xml file. Regardless if targeting your EntityManager usage for transaction-type="RESOURCE_LOCAL" or transaction-type="TRANSACTION".
- It's very difficult to guarantee one or the other will be the only one needed. 
- Often times the JPA Provider itself will require both internally to do various optimisations or other special features.

1)  The _jta-data-source_ should always have it's OpenEJB-specific '_JtaManaged_' property set to '_true_' (this is the default)
2)  The _non-jta-data-source_ should always have it's OpenEJB-specific '_JtaManaged_' property set to '_false_'.

## Be detach aware

A warning for any new JPA user is by default all objects will detach at the end of a transaction. People typically discover this when the go to remove or update an object they fetched previously and get an exception like "You cannot perform operation delete on detached object".

All ejb methods start a transaction unless 
a) you [configure them otherwise](https://tomee.apache.org/transaction-annotations.html) , or 
b) the caller already has a transaction in progress when it calls the bean. If you're in a test case or a servlet, it's most likely B that is biting you. You're asking an ejb for some persistent objects, it uses the EntityManager in the scope of the transaction started around it's method and returns some persistent objects, by the time you get them the transaction has completed and now the objects are detached.

### Solutions

1.  Call EntityManager.merge(..) inside the bean code to reattach your object.
2.  Use PersistenceContextType.EXTENDED as in '@PersistenceContext(unitName = "movie-unit", type = PersistenceContextType.EXTENDED)' for EntityManager refs instead of the default of PersistenceContextType.TRANSACTION.
3.  If testing, use a technique to execute transactions in your test code. That's described here in [Unit testing transactions](https://tomee.apache.org/unit-testing-transactions.html)

### Optional reading
- [JPA 101](https://tomee.apache.org/master/docs/jpa-concepts.html)
- [Tomee And hibernate](https://tomee.apache.org/latest/docs/tomee-and-hibernate.html)
