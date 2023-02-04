
- Download and extract the zip or tar file at some suitable location
- Support that dir is DERBY_HOME (its a full path)

## Client server mode:

### 1) Start Server
- From terminal
- `cd $DERBY_HOME`
- create data directory `mkdir data`
- `cd data`
- start derby server `java -jar ../lib/derbyrun.jar server start`

### 2) Start Command line interface `ij`
- Start new terminal
- `cd $DERBY_HOME/data`
-  `java -jar ../lib/derbyrun.jar ij`

### 3) Create DB using `ij`
- `connect 'dbc:derby://localhost:1527/cardb;create=true';`
- Above command
	1) create=true - will create a DB if not exist
	2) and also connect to that DB

### 4) Use user and password
- In the data directory we created earlier, create file `derby.properties`
- Add following contents
``` text
derby.connection.requireAuthentication=true
derby.authentication.provider=BUILTIN
derby.user.carman=carman123
```
- Restart the server as described in step 1.
- Now start `ij` and to connect to DB using `jdbc:derby://localhost:1527/cardb;user=carman;password=carman123`

### 5) some useful `ij` commands
- show connections;
- show tables;
- quit;

### Useful links
- [Video](https://www.youtube.com/watch?v=L3CRBvaXJLE&list=PLR2yPNIFMlL8e7NP-tsZ-XDfniK0XRS09&index=2)
- https://oglimmer.medium.com/tomee-and-jpa-datasources-b95acb8663e4
- https://tomee.apache.org/tomcat-jpa.html


