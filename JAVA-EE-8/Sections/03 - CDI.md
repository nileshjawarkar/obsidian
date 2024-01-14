CDI (Contexts and Dependency Injection) is a standard dependency injection framework included in Java EE 6 and higher. **Dependency injection** is a design pattern in which an object or function receives other objects or functions that it depends on. A form of inversion of control, dependency injection aims to separate the concerns of constructing objects and using them, leading to loosely coupled programs.

### Bean discovery

File "bean.xml" is used to configure bean discovery mode. Bean discovery mode tells the container how to manage the beans.
- all - Manage all the beans
- annotated (default) - Manages annotated beans only
- none - CDI disabled

Sample beans.xml file -
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
bean-discovery-mode="all">
</beans>
```

*Note* - Save this bean.xml to "WEB-INF" directory of web-app.
### Injection

The _@Inject_ annotation is CDI's actual workhorse. It allows us to define injection points in the client classes.

We can modify "CarManufacturer" class to use CDI.

``` java
package com.nilesh.jawarkar.learn.javaee8.boundry;

import java.util.List;
import javax.inject.Inject;

import com.nilesh.jawarkar.learn.javaee8.control.CarFactory;
import com.nilesh.jawarkar.learn.javaee8.control.TrackColor;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.CarCreated;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;
import com.nilesh.jawarkar.learn.javaee8.entity.InvalidEngine;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

public class CarManufacturer {

    @Inject
	CarFactory        carFactory;
	@Inject
	CarRepository carRepository;

    /* Not required now -
	public CarManufacturer(CarFactory carFactory, 
		CarRepository carRepository) {
		this.carFactory = carFactory;
		this.carRepository = carRepository;
	} */

	public Car createCar(final Specification spec) {
		if (spec.getEngineType() == EngineType.UNKNOWN)
			throw new InvalidEngine();
		final Car car = carFactory.createCar(spec);
		carRepository.save(car);
		carCreatedEvent.fire(new CarCreated(car.getId()));
		return car;
	}

	public Car retrieveCar(final String id) {
		return carRepository.findById(id);
	}

	public List<Car> retrieveCars() {
		return carRepository.getAll(null, null);
	}

	public List<Car> retrieveCars(final String filterByAttr, 
		final String filterByValue) {
		if (filterByAttr == null 
			|| filterByAttr.equals("") 
			|| filterByValue == null)
			return null;
		return retrieveCars();
	}
}
```

### Managing ambiguity

When more than one implementations are available for a type and CDI didn't know which implementation to inject. This is  a ambiguous situation. Solving this ambiguity is easy. CDI, by default, annotates all the implementations of an interface with the _@Default_ annotation. So, we should explicitly tell it which implementation should be injected into the client class my annotating one which you want to be injected by default with _@Default_ and others by annotating with _@Alternative_.

### Life cycle callbacks

- *@PostConstruct* - Method marked with this annotation is called after instance creation
- *@PreDestroy* - Method marked with this annotation is called before destroying the instance


