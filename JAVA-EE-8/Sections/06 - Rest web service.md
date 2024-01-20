-  JAX-RS resources
- JSON-P, JSON-B
- Custom HTTP response
- Validating communication
- Exception handling

### JAX-RS configuration

``` java
package com.nilesh.jawarkar.learn.javaee8.config;

import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("api")
public class JAXRSConfiguration extends Application {
}
```

JAX-RS application has to start with defining root, with which all rest path start. This config class must inherit from "javax.ws.rs.core.Application" and defines application path using annotation "ApplicationPath". Path defined using annotation "ApplicationPath" became root path or start path for web-service.

---

### Define web resource

``` java
package com.nilesh.jawarkar.learn.javaee8.boundry.resource;

import java.util.List;

import javax.inject.Inject;
import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;
import com.nilesh.jawarkar.learn.javaee8.boundry.CarManufacturer;

@Path("v1/cars")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class CarResourceV1 {
	@Inject
	CarManufacturer carManufacturer;
	
	@GET
	public List<Car> getCars() {
		return carManufacturer.retrieveCars();
	}
	
	@GET
	@Path("{id}")
	public Car getCar(@PathParam("id") @DefaultValue("zyx") String id) {
		return carManufacturer.retrieveCar(id);
	}
	
	@POST
	public Car createCar(Specification spec) throws InvalidEngine {
		return carManufacturer.createCar(spec);
	}
}
```

Above class defines "v1/cars" resource . 
- GET ${baseUrl}/api/v1/cars 
	- Get on above url will execute "getCars" method of above class.
	- Return value "List\<car>", will automatically get converted to JSON string using JSON-B.

``` sh
curl "http://localhost:8080/carman/api/v1/cars"

# output
[{"color":"BLUE","engineType":"DIESEL","id":"3794bab4-a59f-47a6-a4fb-da597d46f782"},{"color":"WHITE","engineType":"PETROL","id":"de9f09db-c3da-4697-a3e3-66bbaea0fb7b"},{"color":"BLUE","engineType":"DIESEL","id":"63308dc3-c958-4b22-9503-f75604be5f0f"}]

```

- GET ${baseUrl}/api/v1/cars/3794bab4-a59f-47a6-a4fb-da597d46f782
	- This will retrieve only one car.

``` sh
curl "http://localhost:8080/carman/api/v1/cars/3794bab4-a59f-47a6-a4fb-da597d46f782"

# output
{"color":"BLUE","engineType":"DIESEL","id":"3794bab4-a59f-47a6-a4fb-da597d46f782"}
```

- POST ${baseUrl}/api/v1/cars 
	 - Post on above url will execute "createCar" method and create new car.
	 - Client will get json data as response. This conversion from Car to json will be done automatically by JSON-B.
``` sh
curl "http://localhost:8080/carman/api/v1/cars" -i -XPOST -H 'content-type: Application/json' -d ' {"color": "BLUE", "engineType": "DIESEL"}'

# output
HTTP/1.1 200 
Date: Sat, 20 Jan 2024 07:48:02 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Server: Apache TomEE

{"color":"BLUE","engineType":"DIESEL","id":"77bc3e13-0385-4425-a016-e41cdac32c61"}
```

- Annotation "@Produces(MediaType.APPLICATION_JSON)" & "@Consumes(MediaType.APPLICATION_JSON)" play big role in automatic conversion of JSON to object and object to JSON.

---

### JSON-P, Path Param, Query Param, UriInfo

**JSON-P** : It is used to manage json processing manually.  In following example, input is accepted json objects and we explicitly reading specific properties from json objects. In similar way, we are creating output as a json objects and packaging them in Response  with relevant status. This is very important, when we work with very complex data.

``` java
package com.nnj.learn.javaee8.boundry.resource;

import java.net.URI;
import java.util.List;
import java.util.logging.Logger;
import javax.inject.Inject;
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonObject;
import javax.json.stream.JsonCollectors;
import javax.validation.constraints.NotNull;
import javax.ws.rs.Consumes;
import javax.ws.rs.DefaultValue;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.UriInfo;

import com.nilesh.jawarkar.learn.javaee8.boundry.CarManufacturer;
import com.nilesh.jawarkar.learn.javaee8.boundry.InvalidEngine;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

@Path("v2/cars")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class CarResourceV2 {
	static final Logger LOGGER = Logger.getLogger(CarResourceV2.class.getName());
	
	@Inject
	CarManufacturer carManufacturer;
	
	@Context
	UriInfo uriInfo;
	
	Color defaultColor = Color.RED;
	
	@GET
	public Response getCars(
		@QueryParam("attr") @DefaultValue("") final String filterByAttr,
		@QueryParam("value") @DefaultValue("") final String filterByValue) {
		final List<Car> cars = 
			carManufacturer.retrieveCars(filterByAttr, filterByValue);
		if (cars == null || cars.size() == 0) {
			return Response.noContent().build();
		}
		
		LOGGER.info("Number of cars - " + cars.size());
		JsonArray jsonArray = cars.stream().map(
			car -> Json.createValue(car.getId()))
			.collect(JsonCollectors.toJsonArray());
		return Response.ok().entity(jsonArray).build();
	}
	
	@GET
	@Path("{id}")
	public Response getCar(@PathParam("id") @DefaultValue("zyx") String id) {
		Car car = carManufacturer.retrieveCar(id);
		if (car == null) {
			return Response.noContent().build();
		}
		
		JsonObject jsonObj = Json.createObjectBuilder()
			.add("color", car.getColor().name())
			.add("engineType", car.getEngineType().name())
			.add("id", car.getId()).build();
		return Response.ok().entity(jsonObj).build();
	}
	
	@POST
	public Response createCar(@NotNull final JsonObject jsonSpec) throws InvalidEngine {
	
		final String strColor = (jsonSpec.containsKey("color") 
			? jsonSpec.getString("color") : null);
		final String strEngineType = (jsonSpec.containsKey("engineType") 
			? jsonSpec.getString("engineType") : null);
		
		if (strEngineType != null) {
			final Specification spec = new Specification();
			spec.setColor(strColor == null 
				? defaultColor : Color.valueOf(strColor));
			spec.setEngineType(EngineType.valueOf(strEngineType));
			final Car car = carManufacturer.createCar(spec);
			final URI uri = uriInfo.getBaseUriBuilder()
				.path(CarResourceV2.class)
				.path(CarResourceV2.class, "getCar")
				.build(car.getId());
			return Response.created(uri).entity(car).build();
		}
		return Response.status(Response.Status.BAD_REQUEST).build();
	}
}
```

---

**Path Param** : When user need to retrieve specific object, user will pass specific id as a input. In rest web service, user can pass values as a "path param" or as a "query param".  
- "path param" is more preferred in rest web service

Above example demonstrate use of path param. 
``` sh
curl "http://localhost:8080/carman/api/v2/cars/3794bab4-a59f-47a6-a4fb-da597d46f782" -i

# output
HTTP/1.1 200 
Date: Sat, 20 Jan 2024 08:11:49 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Server: Apache TomEE

{"color":"BLUE","engineType":"DIESEL","id":"3794bab4-a59f-47a6-a4fb-da597d46f782"}
```

**Query Param** :  It is traditional HTTP way of passing values. QueryParam annotation is used toretrive value pass in the http url in following format -
- GET ${baseUrl}/api/v1/cars?attr=color&value=BLUE"
- Using it we can retrieve following key value pairs
	- attr=color
	- value=BLUE

``` sh
curl "http://localhost:8080/carman/api/v2/cars?attr=color&value=BLUE" -i

#output
HTTP/1.1 200 
Date: Sat, 20 Jan 2024 08:20:37 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Server: Apache TomEE

["3794bab4-a59f-47a6-a4fb-da597d46f782"]
```

**UriInfo** : An instance of `UriInfo` can be injected as field or method parameter using the @Context annotation. UriInfo provides access to application and request URI information.

``` java
final URI uri = uriInfo.getBaseUriBuilder()
	.path(CarResourceV2.class)
	.path(CarResourceV2.class, "getCar")
	.build(car.getId());
```

In above example, we are build path as follows -
- http://localhost:8080/carman/api/v2/cars/3794bab4-a59f-47a6-a4fb-da597d46f782
	- http://localhost:8080/carman/api  <- uriInfo.getBaseUriBuilder()
	- /api/v2 <- .path(CarResourceV2.class)
	- /cars/3794bab4-a59f-47a6-a4fb-da597d46f782 <- .path(CarResourceV2.class, "getCar").build(car.getId());

