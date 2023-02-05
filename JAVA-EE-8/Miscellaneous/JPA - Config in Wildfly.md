### 1) Download and extract wildfly at suitable location
- Consider this location as WILDFLY_HOME (its a full path)

### 2) Add user - This user will be used for web - admin console
- `cd $WILDFLY_HOME/bin`
- run `./add-user.sh`
- add-user ask following questions
``` text
What type of user do you wish to add? 
 a) Management User (mgmt-users.properties) 
 b) Application User (application-users.properties)
(a):  

Enter the details of the new user to add.
Using realm 'ManagementRealm' as discovered from the existing property files.
Username : student
Password recommendations are listed below. To modify these restrictions edit the add-user.properties configuration file.
 - The password should be different from the username
 - The password should not be one of the following restricted values {root, admin, administrator}
 - The password should contain at least 8 characters, 1 alphabetic character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
Password : 
WFLYDM0102: Password should have at least 1 non-alphanumeric symbol.
Are you sure you want to use the password entered yes/no? yes
Re-enter Password : 
What groups do you want this user to belong to? (Please enter a comma separated list, or leave blank for none)[  ]: 
About to add user 'student' for realm 'ManagementRealm'
Is this correct yes/no? yes
Added user 'student' to file '/home/nilesh/Downloads/Softwares/wildfly-26.1.2.Final/standalone/configuration/mgmt-users.properties'
Added user 'student' to file '/home/nilesh/Downloads/Softwares/wildfly-26.1.2.Final/domain/configuration/mgmt-users.properties'
Added user 'student' with groups  to file '/home/nilesh/Downloads/Softwares/wildfly-26.1.2.Final/standalone/configuration/mgmt-groups.properties'
Added user 'student' with groups  to file '/home/nilesh/Downloads/Softwares/wildfly-26.1.2.Final/domain/configuration/mgmt-groups.properties'
Is this new user going to be used for one AS process to connect to another AS process? 
e.g. for a slave host controller connecting to the master or for a Remoting connection for server to server Jakarta Enterprise Beans calls.
yes/no? yes
To represent the user add the following to the server-identities definition <secret value="c3R1ZGVudDEyMw==" />
```
- here i created user `student` with password `student123`

### 3) Add datasource using CLI
- Start server (from bin directory)
	`./standalone.sh`

- Start CLI - It will connect to server running on localhost:9990
	`./jboss-cli.sh --connect`

#### MySQL

- Add module - This command will add modules under directory `${WILDFLY_HOME}/modules`
	`module add --name=com.mysql --resources=/home/nilesh/Downloads/Softwares/mysql-connector-java-8.0.28.jar --dependencies=javax.api,javax.transaction.api`

- Register as JDBC driver
	`/subsystem=datasources/jdbc-driver=mysql:add(driver-name="mysql",driver-module-name="com.mysql",driver-class-name=com.mysql.cj.jdbc.Driver)`

- Add datasource
	`data-source add --jndi-name=java:jboss/datasources/MysqlDS01 --name=MysqlPool --connection-url=jdbc:mysql://myhost1:3306,myhost2:3307/db_name --driver-name=mysql --user-name=test --password=test123`

#### Derby

- Add module 
	`module add --name=org.apache.derby2 --resource-delimiter=, --resources=/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyclient.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_cs.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_de_DE.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_es.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_fr.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_hu.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_it.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_ja_JP.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_ko_KR.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_pl.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_pt_BR.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_ru.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_zh_CN.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyLocale_zh_TW.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbyshared.jar,/home/nilesh/Downloads/Softwares/db-derby-10.16.1.1-bin/lib/derbytools.jar --dependencies=javax.api,javax.transaction.api`

- To add multiple jar as a module resource, we need to define the resource delimiter using `--resource-delimiter=,` . Now use this delimiter to add multiple jars as shown in above command.
- Above command will generate module with following module.xml
``` xml
<?xml version="1.0" ?>
<module xmlns="urn:jboss:module:1.1" name="org.apache.derby">
    <resources>
        <resource-root path="derbyclient.jar"/>
        <resource-root path="derbyLocale_cs.jar"/>
        <resource-root path="derbyLocale_de_DE.jar"/>
        <resource-root path="derbyLocale_es.jar"/>
        <resource-root path="derbyLocale_fr.jar"/>
        <resource-root path="derbyLocale_hu.jar"/>
        <resource-root path="derbyLocale_it.jar"/>
        <resource-root path="derbyLocale_ja_JP.jar"/>
        <resource-root path="derbyLocale_ko_KR.jar"/>
        <resource-root path="derbyLocale_pl.jar"/>
        <resource-root path="derbyLocale_pt_BR.jar"/>
        <resource-root path="derbyLocale_ru.jar"/>
        <resource-root path="derbyLocale_zh_CN.jar"/>
        <resource-root path="derbyLocale_zh_TW.jar"/>
        <resource-root path="derbyshared.jar"/>
        <resource-root path="derbytools.jar"/>
    </resources>
    <dependencies>
        <module name="javax.api"/>
        <module name="javax.transaction.api"/>
    </dependencies>
</module>
```
- Register as JDBC driver
	`/subsystem=datasources/jdbc-driver=derby:add(driver-name="derby",driver-module-name="org.apache.derby",driver-class-name=org.apache.derby.jdbc.ClientDriver)`

- Add datasource
	`data-source add --jndi-name=java:jdbc/carman --name=DerbyPool --connection-url=jdbc:derby://localhost:1527/cardb;create=true --driver-name=derby --user-name=carman --password=carman123`

### 4) Use datasource in persistence.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
	xmlns="http://xmlns.jcp.org/xml/ns/persistence"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
<persistence-unit name="prod" transaction-type="JTA">
	<jta-data-source>jdbc/carman</jta-data-source>
	<exclude-unlisted-classes>false</exclude-unlisted-classes>
	<properties>
		<property
			name="javax.persistence.schema-generation.database.action"
			value="drop-and-create" />
		</properties>
</persistence-unit>
</persistence>
```

### 5) Useful links
- [Video - add datasource - 1](https://www.youtube.com/watch?v=I8t1TLSeEBw)
- [Video - add datasource - 2](https://www.youtube.com/watch?v=xSHXMcRsF0A)
- [Add data source using CLI](http://www.mastertheboss.com/jbossas/jboss-datasource/how-to-configure-a-datasource-with-jboss-7/)
- [JDBC url for multiple database](https://www.baeldung.com/java-jdbc-url-format)
- [Web side - master the boss](http://www.mastertheboss.com/)
- 
