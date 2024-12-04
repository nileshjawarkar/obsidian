
### Enable SSO in server.xml - Uncomment following line

``` xml
<Valve className="org.apache.catalina.authenticator.SingleSignOn" />
```

### Configure Realm - Realm basically controls, the way credentials repository is used to validate the user.

In this example, we will be using default realm, which uses tomcat-users.xml as credential repository.

``` xml
<Realm className="org.apache.catalina.realm.UserDatabaseRealm"
    resourceName="UserDatabase"/>
<GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
</GlobalNamingResources>
```

### Add role and user to the tomcat-users.xml

``` xml
<role rolename="app-admin" />
<user username="user1" password="user1123" roles="app-admin" />
```

### Now configure your webapp using web.xml to use this user/role

``` xml
   <security-constraint>
      <web-resource-collection>
         <web-resource-name>My App</web-resource-name>
         <url-pattern>/api/*</url-pattern>
         <http-method>GET</http-method>
         <http-method>POST</http-method>
      </web-resource-collection>
      <auth-constraint>
         <role-name>app-admin</role-name>
      </auth-constraint>
      <user-data-constraint>
         <transport-guarantee>NONE</transport-guarantee>
      </user-data-constraint>
   </security-constraint>
   <login-config>
      <auth-method>BASIC</auth-method>
   </login-config>
```

To configure the app 
- We need to define security constraint with the url to protect and role defined in the realm.
- Define login config with auth method.. in this case BASIC.

### Use curl to access "localhost:8080/api/prj1/cars"

- Encode use and password using base64 encoding
``` sh
echo 'user1:user1123' | base64

# output

dXNlcjE6dXNlcjExMjMK
```

- Access protected API - 1st time

``` sh
curl "http://localhost:8080/prj1/api/cars" -i -XGET -H 'Authorization: Basic dXNlcjE6dXNlcjExMjMK'

# output

HTTP/1.1 200 
Cache-Control: private
Set-Cookie: JSESSIONIDSSO=DFA4BDC68E14F0EA34558B0D907A5D3A; Path=/; HttpOnly
Set-Cookie: JSESSIONID=9728B774D7E35C916B6A5CBEA4F2AFEC; Path=/prj1; HttpOnly
Date: Wed, 04 Dec 2024 05:16:44 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Server: Apache TomEE

[{"color":"BLACK","engineType":"DIESEL","identifier":"611cbd91-d2e4-4c16-b2f4-a9abb0de4614"},{"color":"RED","engineType":"PETROL","identifier":"6d4a5911-881a-4f03-b27d-dce02ec37f2f"}]
```

- Access same API second time using cookies -

``` sh
curl "http://localhost:8080/prj1/api/cars" -i -XGET --cookie "JSESSIONIDSSO=DFA4BDC68E14F0EA34558B0D907A5D3A; Path=/; HttpOnly"

# output

HTTP/1.1 200 
Cache-Control: private
Date: Wed, 04 Dec 2024 05:19:08 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Server: Apache TomEE

[{"color":"BLACK","engineType":"DIESEL","identifier":"611cbd91-d2e4-4c16-b2f4-a9abb0de4614"},{"color":"RED","engineType":"PETROL","identifier":"6d4a5911-881a-4f03-b27d-dce02ec37f2f"}]
```