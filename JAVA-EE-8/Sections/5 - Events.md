 ***Synchronous CDI events***
 ***Implemented using : Event<\T> and @Observes***
 
- Even description class 
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

- Producer
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
- Listener
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

