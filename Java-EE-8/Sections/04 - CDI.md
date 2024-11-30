- CDI (Contexts and Dependency Injection) is a standard dependency injection framework included in JavaEE 6 and higher. 
- **Dependency injection** is a design pattern in which an object or function receives other objects or functions that it depends on. 
- A form of inversion of control, dependency injection aims to separate the concerns of constructing objects and using them, leading to loosely coupled programs.

### Now modify our code to use CDI

The _@Inject_ annotation is CDI's actual workhorse. It allows us to define injection points in the client classes.

We can modify "CarManufacturer" class to use CDI and inject "CarFactory" and "CarRepository". But how sever knows to inject which instance - we will discuss it in next section.

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
		return carRepository.getAll(filterByAttr, filterByValue);
	}
}
```

### Bean discovery mode - controls the behaviour of CDI in JavaEE and con be configured using "beans.xml"

- Bean discovery mode tells the container how to manage the beans. In simple words, bean discovery mode defines which bean can be injected.
	- By default, only "annotated" bean can be injected. It means only those bean can be injected, which are marked with some CDI or EJB annotations. We will go through these annotation in separate section "Scopes".
	- all - All bean can be injected.
	- none - CDI disabled

 In our code example, CarRepository and CarFactory didn't have any annotation on it. So to make its injection possible, we need to set discovery mode to "all" using beans.xml as shown below. 

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
bean-discovery-mode="all">
</beans>
```

*Note* - Save this bean.xml to "WEB-INF" directory of web-app.
### Managing ambiguity

- Currently in our project "CarRepository" interface has only one implementation. It is basically in memory store. It means when we restart our app, data hold by this implementation will be lost.  But in coming days when we start exploring persistence, we will create new implementation of "CarRepository", which will write data to some DB.

- But at that time we will have 2 implementation of "CarRepository", this will lead to ambiguity. When more than one implementations are available for a type and CDI didn't know which implementation to inject. 

- Solving this ambiguity is easy. CDI, by default, annotates all the implementations of an interface with the _@Default_ annotation. So, we should explicitly tell it which implementation should be injected into the client class. May be annotating one which you want to be injected by default with _@Default_ and others by annotating with _@Alternative_.

- We will discuss this again in persistence section.
- We can also use Producers and Qualifiers for the same purpose. We will discuss them latter in separate section.

### Life cycle callbacks

- *@PostConstruct* - Method marked with this annotation is called after instance creation
- *@PreDestroy* - Method marked with this annotation is called before destroying the instance

As a exmaple, lets modify implementation of "CarRepositoryInMemImpl" to use these annotations -

``` java
package com.nnj.learn.javaee8.control;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.logging.Logger;
import java.util.stream.Collectors;

import com.nnj.learn.javaee8.entity.Car;
import com.nnj.learn.javaee8.entity.Color;
import com.nnj.learn.javaee8.entity.EngineType;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;

public class CarRepositoryInMemImpl implements CarRepository {
	private List<Car> list = new ArrayList<>();
	
	static final Logger LOGGER = Logger.getLogger(CarRepositoryInMemImpl.class.getName());

	//public CarRepositoryInMemImpl() {
	@PostConstruct
	public void Init() {
		final Car c1 = new Car();
		c1.setId("3794bab4-a59f-47a6-a4fb-da597d46f782");
		c1.setColor(Color.BLUE);
		c1.setEngine(EngineType.DIESEL);
		list.add(c1);

		final Car c2 = new Car();
		c2.setId(UUID.randomUUID().toString());
		c2.setColor(Color.WHITE);
		c2.setEngine(EngineType.PETROL);
		list.add(c2);
		
		LOGGER.info("Init called using @PostConstruct.");
	}
	
	@PreDestroy
	public void clean() {
		list.clear();
	}

	@Override
	public Car findById(final String id) {
		for (final Car c : list) {
			final String cid = c.getId();
			if (cid != null & cid.equals(id))
				return c;
		}
		return null;
	}

	@Override
	public List<Car> getAll(final String filterByAttr, final String    filterByValue) {
		if (filterByAttr == null || filterByAttr.equals("") || filterByValue == null || filterByValue.equals(""))
			return list;

		return list.stream().filter(c -> {
			if (filterByAttr.equals("color"))
				return c.getColor() == Color.valueOf(filterByValue);
			if (filterByAttr.equals("engineType"))
				return c.getEngineType() == EngineType.valueOf(filterByValue);
			return false;
		}).collect(Collectors.toList());
	}

	@Override
	public void save(final Car car) {
		list.add(car);
	}
}
```


