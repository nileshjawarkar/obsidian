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

@ApplicationPath("resources")
public class JAXRSConfiguration extends Application {

}
```

JAX-RS application has to start with defining root, with which all rest path start. This config class must inherit from "javax.ws.rs.core.Application" and defines application path using annotation "ApplicationPath". Path defined using annotation "ApplicationPath" became root path or start path for web-service.

---

### Define web resource

``` java
package com.nilesh.jawarkar.learn.javaee8.boundry;

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

@Path("cars")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class CarResource {

	@Inject
	CarManufacturer carManufacturer;

	@POST
	public Car createCar(Specification specs) {
		return carManufacturer.createCar(specs);
	}

	@GET
	public List<Car> retrieveCars() {
		return carManufacturer.retrieveCars();
	}
}
```

Above class defines "cars" resource . 
- GET ${baseUrl}/resources/cars 
	- Get on above url will execute "retrieveCars" method of above class.
	- Return value "List\<car>", will automatically get converted to JSON string using JSON-B.
- POST ${baseUrl}/resources/cars 
	 - Post on above url will execute "createCar" method and create new car.
	 - Client will sent json data using http and this json data will get converted automatically to "Specification" using JSON-B.
- Annotation "@Produces(MediaType.APPLICATION_JSON)" & "@Consumes(MediaType.APPLICATION_JSON)" play big role in automatic conversion of JSON to object and object to JSON.

---

### JSON-P, Path Param, Query Param, UriInfo

**JSON-P** : It is used to manage json processing manually.  In following example, input is accepted json objects and we explicitly reading specific properties from json objects. In similar way, we are creating output as a json objects and packaging them in Response  with relevant status. This is very important, when we work with very complex data.

``` java
package com.nilesh.jawarkar.learn.javaee8.boundry;

import java.net.URI;
import java.util.List;
import javax.inject.Inject;
import javax.json.Json;
import javax.json.JsonArray;
import javax.json.JsonObject;
import javax.json.stream.JsonCollectors;
import javax.validation.constraints.NotNull;

import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.UriInfo;

import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

@Path("/v2/cars")
public class CarResource2 {
	@Context
	UriInfo uriInfo;
	@Inject
	CarManufacturer carManufacturer;
	
	@POST
	public Response createCar(@NotNull final JsonObject jsonSpec) {
		final String strColor = (jsonSpec.containsKey("color")
			? jsonSpec.getString("color")
			: null);
		final String strEngineType = (jsonSpec.containsKey("engineType")
			? jsonSpec.getString("engineType")
			: null);
		
		if (strEngineType != null) {
			final Specification spec = new Specification();
			spec.setColor(strColor == null ? Color.RED : Color.valueOf(strColor));
			spec.setEngineType(EngineType.valueOf(strEngineType));
			final Car car = carManufacturer.createCar(spec);
			final URI uri = uriInfo.getBaseUriBuilder().path(CarResource2.class)
				.path(CarResource2.class, "retrieveCar").build(car.getId());
			return Response.created(uri).entity(car).build();
		}
	
		return Response.status(Response.Status.BAD_REQUEST).build();
	}
	
	@GET
	@Path("{id}")
	public Response retrieveCar(@PathParam("id") final String id) {
		final Car car = carManufacturer.retrieveCar(id);
		if (car != null) {
			final JsonObject jobj = Json.createObjectBuilder()
				.add("id", car.getId())
				.add("color", car.getColor().name())
				.add("engineType", car.getEngineType().name())
				.build();
			return Response.status(Response.Status.OK).entity(jobj).build();
		}
		return Response.status(Response.Status.NO_CONTENT).build();
	}
	
	@GET
	public Response retrieveCars(@QueryParam("attr") final String filterByAttr,
	@QueryParam("value") final String filterByValue) {
		final List<Car> list = carManufacturer.retrieveCars(
			filterByAttr, filterByValue);
		if (list != null) {
			final JsonArray resultList = list.stream()
				.map(c -> Json.createObjectBuilder().add("id", c.getId())
				.add("color", c.getColor().name())
				.add("engineType", c.getEngineType().name()).build())
				.collect(JsonCollectors.toJsonArray());
			return Response.ok().entity(resultList).build();
		}
		return Response.status(Response.Status.NO_CONTENT).build();
	}
}
```

---

**Path Param** : When user need to retrieve specific object, user will pass specific id as a input. In rest web service, user can pass object to find as a "query param" or "path param".  Above example demonstrate use of path param. 

``` sh
curl "http://localhost:8080/carman/resources/v2/cars/f6ed1533-1a83-4a95-9be4-cded938b082a" -i
```

**Query Param** : TODO

``` sh
curl "http://localhost:8080/carman/resources/v2/cars?attr=color&value=BLUE" -i
```

**UriInfo** : TODO
