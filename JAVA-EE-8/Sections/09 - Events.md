This is one of the most powerful feature of JavaEE.  It helps to develop decoupled application as component can send data to another component without any form of relation between them.

- Even description class (payload)
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

- Producer "CarManufacturer" - "createCar" method will fire the event.
``` java
@Stateless
public class CarManufacturer {
	@Inject
	CarFactory carFactory;
	@Inject CarRepository carRepository;
	
	@Inject
	Event<CarCreated> carCreatedEvent;
	
	public Car createCar(final Specification spec) {
		if (spec.getEngineType() == EngineType.UNKNOWN)
			throw new InvalidEngine();
		
		final Car car = carFactory.createCar(spec);
		carRepository.save(car);
		
		carCreatedEvent.fire(new CarCreated(car.getId()));
		return car;
	}
}
```

- Listener - This class will handle the fired event.
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import javax.enterprise.event.Observes;
import com.nilesh.jawarkar.learn.javaee8.entity.CarCreated;

public class CarCreationListener {
	public void onCarCreation(@Observes CarCreated event) {
		System.out.println("Car created - " + event.getIdentifier());
	}
}

```


### Qualified events

- Qualifier - Created new qualifier _@NewTech_ for handling creation of ELECTRIC cars.
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import javax.inject.Qualifier;

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, 
   ElementType.PARAMETER })
public @interface NewTech {
}
```

- Producer - "createCar" method will fire "newTechCarCreatedEvent" on creation of electric car, otherwise it will fire "carCreatedEvent". Note that  "newTechCarCreatedEvent" is decorated with _@NewTech_ qualifier.
``` java
@Inject
Event<CarCreated> carCreatedEvent;

@NewTech
@Inject
Event<CarCreated> newTechCarCreatedEvent;

@Resource
ManagedExecutorService mes;

@TrackColor(Color.ANY)
public Car createCar(final Specification spec) {
	if (spec.getColor() == Color.ANY) {
		throw new InvalidColor(Color.ANY);
	}
	final Car car = this.carFactory.createCar(spec);
	this.entityManager.persist(car);
	if (car.getEngineType() == EngineType.ELECTRIC) {
		this.newTechCarCreatedEvent.fire(new CarCreated(car.getId()));
	} else {
		this.carCreatedEvent.fire(new CarCreated(car.getId()));
	}
	
	this.mes.execute(() -> this.carProcessing.process(car));
	return car;
}
```

- Handler OR Consumer - New method _onNewTechCarCreation_ is added to handle "newTechCarCreatedEvent". Note this handler is using _@NewTech_ qualifier.
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.concurrent.locks.LockSupport;
import javax.enterprise.event.ObservesAsync;
import com.nilesh.jawarkar.learn.javaee8.entity.CarCreated;

public class CarCreationListener {
	public void onCarCreation(@Observes final CarCreated event) {
		LockSupport.parkNanos(2000000000L);
		System.out.println("Car created - " + event.getIdentifier());
	}
	
	public void onNewTechCarCreation(@Observes @NewTech final CarCreated event) 
	{
		LockSupport.parkNanos(2000000000L);
		System.out.println("New Tech Car created - " + event.getIdentifier());
	}
}

```

### Conditional events (TODO) ...
