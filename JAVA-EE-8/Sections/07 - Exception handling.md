This section discuss the way, using which we can manage exceptions thrown during rest web-service calls. JavaEE provides a mechanism, which will handle the exception thrown and convert it to appropriate response. Let discuss it using following example -

- CarResourceV2 -  CarManufacturers "createCar"  API will throw InvalidEngine exception, if user provided un-supported engine type as a input -
``` java
@POST
public Response createCar(@NotNull final JsonObject jsonSpec) {
	final String strColor = (jsonSpec.containsKey("color") 
		? jsonSpec.getString("color") : null);
	final String strEngineType = (jsonSpec.containsKey("engineType") 
		? jsonSpec.getString("engineType") : null);
	
	if (strEngineType != null) {
		final Specification spec = new Specification();
		EngineType engineType = EngineType.UNKNOWN;
		try {
			engineType = EngineType.valueOf(strEngineType);
		} catch (Exception e) {
			LOGGER.severe("Engine type - \'" + strEngineType 
				+ "\' not valid.");
		}
		spec.setEngineType(engineType);
		spec.setColor(strColor == null 
			? defaultColor : Color.valueOf(strColor));
		
		final Car car = carManufacturer.createCar(spec);
		final URI uri = uriInfo.getBaseUriBuilder()
			.path(CarResourceV2.class)
			.path(CarResourceV2.class, "getCar")
			.build(car.getId());
		return Response.created(uri).entity(car).build();
	}
	return Response.status(Response.Status.BAD_REQUEST).build();
}
```

- JavaEE provide "ExceptionMapper" interface and "Provider" annotation, using them we have implement a handler. This handler will be used automatically by JavaEE, to handle the exception thrown.
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import com.nilesh.jawarkar.learn.javaee8.entity.InvalidEngine;
import javax.json.Json;
import javax.ws.rs.core.Response;
import javax.ws.rs.ext.ExceptionMapper;
import javax.ws.rs.ext.Provider;

@Provider
public class InvalidEngineExceptionMapper implements ExceptionMapper<InvalidEngine>{
	@Override
	public Response toResponse(InvalidEngine exception) {
		return Response.status(Response.Status.BAD_REQUEST)
		.header("X-car-error", exception.getMessage())
		.entity(Json.createObjectBuilder()
			.add("error_message", exception.getMessage())
			.add("status_code", Response.Status.BAD_REQUEST.getStatusCode())
			.build())
		.build();
	}
}
```

- Mark "InvalidEngine" exception as Application exception, to avoid EJB container to wrap it using EJB exception (otherwise we can see this exception in logs).
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

import javax.ejb.ApplicationException;

@ApplicationException
public class InvalidEngine extends RuntimeException {
	private static final long serialVersionUID = 1L;
	public InvalidEngine(String message) {
		super(message);
	}
}
```