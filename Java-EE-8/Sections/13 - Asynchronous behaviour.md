
### a) Asynchronous EJB

- On web service invocation, many times we want to execute some job on the server, which may take time. And also response to the customer may not depend on this job. In such case, longer wait time for the customer will not be justified and will impact the experience.
- To execute such job, we can use Asynchronous EJB.
- Consider a example of create car api where we want to process the car after its creation. This car processing may take time. In following example, we will how this car processing is managed using asynchronous EJB.
- Create stateless EJB -
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.concurrent.locks.LockSupport;
import javax.ejb.Asynchronous;
import javax.ejb.Stateless;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;

@Stateless
public class CarProcessing {

	@Asynchronous
	public void process(final Car car) {
		System.out.println(">> START - Processing a Car - " + car.getId() + " >>");
		// -- This is to simulate heavy job that takes 5 seconds to process.
		LockSupport.parkNanos(5000000000L);
		System.out.println("<< END - " + car.getId() + " <<");
	}
}
```
- Call the asynchronous method of the EJB
``` java
@Inject
CarProcessing carProcessing;

@TrackColor(Color.ANY)
public Car createCar(final Specification spec) {
	if (spec.getColor() == Color.ANY) {
		throw new InvalidColor(Color.ANY);
	}
	
	final Car car = this.carFactory.createCar(spec);
	this.entityManager.persist(car);
	this.carCreatedEvent.fire(new CarCreated(car.getId()));
	
	//-- Heavy car processing job - executed using Asynchronous EJB
	this.carProcessing.process(car);
	return car;
}

```
- Limitation of Asynchronous EJB
	1) Can not be used with CDI beans
	2) Using Asynchronous with statefull EJB will be problematic, specifically when job takes longer to complete and user session expired before it. So it usage best suit to stateless EJB.

### b) Asynchronous CDI Events

- We already seen synchronous events, in this section we will modify same example to demonstrate use of asynchronous events.
1) Even description class (No modification)
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

public class CarCreated {
	private final String identifier;
	
	public CarCreated(String identifier) {
		this.identifier = identifier;
	}
	
	public String getIdentifier() {
		return identifier;
	}
}
```

2) Producer "CarManufacturer" - "fireAsync" method used to fire event.
``` java
@Inject
Event<CarCreated> carCreatedEvent;

@TrackColor(Color.ANY)
public Car createCar(final Specification spec) {
	if (spec.getColor() == Color.ANY) {
		throw new InvalidColor(Color.ANY);
	}
	
	final Car car = this.carFactory.createCar(spec);
	this.entityManager.persist(car);
	this.carCreatedEvent.fireAsync(new CarCreated(car.getId()));
	return car;
}
```

4) Listener - Now "onCarCreation" method uses "@ObservesAsync" annotation
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.concurrent.locks.LockSupport;
import javax.enterprise.event.ObservesAsync;
import com.nilesh.jawarkar.learn.javaee8.entity.CarCreated;

public class CarCreationListener {
	public void onCarCreation(@ObservesAsync final CarCreated event) {
		System.out.println(">> START - Processing a event - " + 
		          event.getIdentifier() + " >>");
		//-- Added some sleep to demonstrate heavy job.
		LockSupport.parkNanos(2000000000L);
		System.out.println("<< END - Processing a event - " + 
		          event.getIdentifier() + " <<");
	}
}
```

### c) Managed threads

- In JAVA EE, we must not start our own thread.
- All the treading should be managed by container.
- So that we will not end up creating resource leak, when container shutdown.
- If there is need to create thread explicitly - we should use "ManagedExecutorService".

1) Modified "CarProcessing" class we used in previous section and now it contains one Asynchronous "processAsync" and one Synchronous "process" method.
2) We will use "ManagedExecutorService" to execute Synchronous method asynchronously.
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.concurrent.locks.LockSupport;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;

import javax.ejb.Asynchronous;
import javax.ejb.Stateless;

@Stateless
public class CarProcessing {

	@Asynchronous
	public void processAsync(final Car car) {
		System.out.println(">> START - Processing a Car - " + car.getId() + " >>");
		// -- This is to simulate heavy job that takes 5 seconds to process.
		LockSupport.parkNanos(5000000000L);
		System.out.println("<< END - " + car.getId() + " <<");
	
	}
	
	public void process(final Car car) {
		System.out.println(">> START - Sync - Processing a Car - " 
		                              + car.getId() + " >>");
		// -- This is to simulate heavy job that takes 5 seconds to process.
		LockSupport.parkNanos(5000000000L);
		System.out.println("<< END - " + car.getId() + " <<");
	}
}
```

2) In class "CarManufacturer" - we injected "ManagedExecutorService" using "@Resource" annotation. We can see how "carProcessing.process"  is executed using "ManagedExecutorService" and lambda.
``` java
@Resource
ManagedExecutorService mes;

@TrackColor(Color.ANY)
public Car createCar(final Specification spec) {
	if (spec.getColor() == Color.ANY) {
		throw new InvalidColor(Color.ANY);
	}
	
	final Car car = this.carFactory.createCar(spec);
	this.entityManager.persist(car);
	this.carCreatedEvent.fireAsync(new CarCreated(car.getId()));
	
	// -- this.carProcessing.processAsync(car);
	this.mes.execute(() -> this.carProcessing.process(car));
	return car;
}
```

### d) Timers

- Timer are used to execute some job continuously at specific time interval.
- Consider in our application, we have class "CarCache" which cache the cars and "CarManufacturer::retrieveCar" method use this class to retrieve the cars. To keep our car cache up to date, we need load the car in cache at some periodic interval.
- This is where timer are very useful, to execute a job at some periodic interval.
- In Java EE, we can do this using 2 ways..

1) **Using - EJB  "@Schedule" annotation**
	- This can be used with EJB only
	- Preferably used with @Singleton and  @Startup 
	- If used with Stateless EJB - remember container keep pool of instances of stateless type. In short we will end in executing "loadCars" method multiple times (number is depends on number of instances in pool).
	- Without @Startup - execution will be scheduled after first call to  "CarCache" EJB.
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;

import javax.ejb.Schedule;
import javax.ejb.Singleton;
import javax.ejb.Startup;

@Singleton
@Startup
public class CarCache {
	@PersistenceContext(name = "prod")
	EntityManager entityManager;
	HashMap<String, Car> carMap = new HashMap<>();

	//-- Execute loadCars once per hour.
	@Schedule(hour = "*")
	public void loadCars() {
		this.carMap.clear();
		List<Car> list = this.entityManager.createNamedQuery(
			Car.FIND_ALL, Car.class).getResultList();
		list.forEach((car) -> {
			this.carMap.put(car.getId(), car);
		});
	}
	
	public List<Car> retrieveCars() {
		List<Car> list = new ArrayList<>();
		this.carMap.forEach((id, car) -> {
			list.add(car);
		});
		return list;
	}
}
```

2) **Using "ManagedScheduledExecutorService"**

``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.concurrent.TimeUnit;
import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import javax.ejb.Singleton;
import javax.ejb.Startup;
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;

import javax.enterprise.concurrent.ManagedScheduledExecutorService;

@Singleton
@Startup
public class CarCache {
	@Resource
	ManagedScheduledExecutorService mses;
	
	@PersistenceContext(name = "prod")
	EntityManager entityManager;
	
	HashMap<String, Car> carMap = new HashMap<>();
	
	// -- @Schedule(hour = "*")
	public void loadCars() {
		this.carMap.clear();
		List<Car> list = this.entityManager.createNamedQuery(
			Car.FIND_ALL, Car.class).getResultList();
		
		list.forEach((car) -> {
			this.carMap.put(car.getId(), car);
		});
	}
	
	@PostConstruct
	public void scheduleLoading() {
		//-- Executes continuously after delay of 10 seconds and per 60 seconds.
		this.mses.scheduleAtFixedRate(() -> loadCars(), 10, 60, TimeUnit.SECONDS);
	
		//-- Execute only once after delay of 10 seconds
		//-- this.mses.schedule(() -> loadCars(), 10, TimeUnit.SECONDS);
	}
	
	public List<Car> retrieveCars() {
		List<Car> list = new ArrayList<>();
		this.carMap.forEach((id, car) -> {
			list.add(car);
		});
		return list;
	}
}
```

- "ManagedScheduledExecutorService" can also be used schedule job only once after some delay. This is shown at line 44 in above example.

### e) Asynchronous JAX-RS resource

- When asynchronous behaviour is not used - all the api execution done in request thread pool.
- When we want to move api execution out off event thread, we can do it in following ways...

1) **Using Asynchronous EJB** - Just need to make sure resource class is EJB and mark api/method with @Asynchronous annotation.
2) **Using AsyncResponse** and **CompletionStage** - In following example, we will see usage of the both. Please note that calling these async-api's from the client, will work as it is, no specific change is required.

``` java
package com.nnj.learn.javaee8.rest;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CompletionStage;
import java.util.logging.Logger;
import javax.annotation.Resource;
import javax.enterprise.concurrent.ManagedExecutorService;
import javax.inject.Inject;
import javax.json.*;
import javax.json.stream.JsonCollectors;
import javax.validation.constraints.NotNull;

import javax.ws.rs.*;
import javax.ws.rs.container.AsyncResponse;
import javax.ws.rs.container.Suspended;
import javax.ws.rs.core.*;

import com.nnj.learn.javaee8.boundry.CarManufacturer;
import com.nnj.learn.javaee8.entity.*;

@Path("v3/cars")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class CarResourceV3 {
	static final Logger LOGGER = 
		Logger.getLogger(CarResourceV3.class.getName());
	
	@Inject
	CarManufacturer carManufacturer;
	@Context
	UriInfo uriInfo;
	@Inject
	Color defaultColor;
	@Resource
	ManagedExecutorService mes;
	
	@GET
	public CompletionStage<Response> getCars(@QueryParam("attr") 
		@DefaultValue("") final String filterByAttr,
		@QueryParam("value") @DefaultValue("") final String filterByValue) {
		return CompletableFuture.supplyAsync(() -> {
			final List<Car> cars = carManufacturer.retrieveCars(
				filterByAttr, filterByValue);
			if (cars == null || cars.size() == 0) {
				return Response.noContent().build();
			}
			
			LOGGER.info("Number of cars - " + cars.size());
			JsonArray jsonArray = cars.stream().map(car -> 
				Json.createValue(car.getId()))
			.collect(JsonCollectors.toJsonArray());
			return Response.ok().entity(jsonArray).build();
		}, mes);
	}
	
	@GET
	@Path("{id}")
	public Response getCar(@PathParam("id") @DefaultValue("zyx") String id) 
	{
		Car car = carManufacturer.retrieveCar(id);
		if (car == null) {
			return Response.noContent().build();
		}
		JsonObject jsonObj = Json.createObjectBuilder()
			.add("color", car.getColor().name())
			.add("engineType", car.getEngineType().name())
			.add("id", car.getId())
			.build();
		return Response.ok().entity(jsonObj).build();
	}
		
	@POST
	public void createCar(@NotNull final JsonObject jsonSpec, 
		@Suspended AsyncResponse response) {
		//-- Inside MES thread we can not access uriInfo. So creating a
		//-- carUrl here and passing it as arg. Latter IIDD will be replaced
		//-- by actual car-id.
		final String tmpCarUrl = uriInfo.getBaseUriBuilder()
			.path(CarResourceV3.class)
			.path(CarResourceV3.class, "getCar")
			.build("CARID").toString();
			
		mes.execute(() -> {
			response.resume(createCar(jsonSpec, tmpCarUrl));
		});
	}
	
	Response createCar(JsonObject jsonSpec, final String tmpCarUrl) {
		final String strColor = (jsonSpec.containsKey("color") ? 
			jsonSpec.getString("color") : null);
		final String strEngineType = (jsonSpec.containsKey("engineType") ?
			jsonSpec.getString("engineType") : null);
		
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
			spec.setColor(strColor == null ? defaultColor : 
				Color.valueOf(strColor));
			final Car car = carManufacturer.createCar(spec);
			String newCarURL = tmpCarUrl.replace("CARID", car.getId());
			return Response.status(Response.Status.CREATED)
				.header("location", 
					newCarURL).entity(car).build();
		}
		return Response.status(Response.Status.BAD_REQUEST).build();
	}
}
```

**Note** -
 - On calling "response.resume" execution resumes. In our case, we called it using "ManagedExecutorService" so execution will done in separate thread pool ,specific to "ManagedExecutorService".
 - "CompletableFuture.supplyAsync" - by default JDK threads will be used, which is prohibited by Java EE specs. In our case we supplied "ManagedExecutorService" as a second argument to avoid the same.