Scope defines the duration within which bean hold its state. In other words scope defines life cycle for the beans.

### CDI scopes
- _@Dependant_ - default. It means bean life-cycle depends on the bean in which it is injected.
- _@RequestScoped_ - Bean with this scope, will created and destroyed per request from user.
- _@SessionScoped_ - Bean with this scope, will live with the current user session and destroyed when session ends.
- _@ApplicationScoped_ - Bean with this scope, will be created once in application life-cycle and shared with all the sessions.

### EJB scopes
- _@Stateless_ bean - Container manages pull of these beans. so we should not add any state to it. We didn't know which bean we will get from pull.
- _@Statefull_ bean - These bean available to user session, so we can add user specific data to it.
- _@Singleton_ bean - This bean is created 1  for application and available for all users. Each method is synchronised.

*When to use CDI scope annotations and when to EJB annotations? EJB annotations will provide many features (such as transaction management) as compare  to CDI annotations, so in turn it makes them a little heavy. When you don't need these features, in that case you can go for CDI annotations. If needed on CDI beans, you can enable features like transactions using specific annotations (like @Transactional).*

In case of out current implementation, lets discuss some classes and apply scope annotation to them -

**CarRepository** -

- We want only once instance of class "CarRepository", so that can it can share its state with other sessions.
- If we use, CDI session scope, data created by one user will not available for other user.
- So it means we have 2 options, CDI ApplicationScoped annotation OR EJB Singleton annotation. One problem with ApplicationScoped annotation, by default it will not provide any concurrency management but Singleton will provide it. In case of ApplicationScoped annotation, we have to explicitly manage it.
- In addition to concurrency management, singleton also provide transaction management, which is also a required feature for our "CarRepository".
- So singleton is best suited to our need.
- But remember, it need to be applied on implementation "CarRepositoryInMemImpl"

``` java
package com.nnj.learn.javaee8.control;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.logging.Logger;
import java.util.stream.Collectors;

import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import jakarta.ejb.Singleton;

@Singleton
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
	public List<Car> getAll(final String filterByAttr, final String filterByValue) {
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

*In above code we used @PostConstruct, @PreDestroy annotations. These are life-cycle related annotations and will execute that marked method during the specific life-cycle phase of the bean*

**CarManufacturer** -

- This class didn't store any state.
- But responsible for creating and saving the Cars, so it requires, transaction management support.
- Stateless EJB is best suited for the purpose.

``` java
package com.nnj.learn.javaee8.boundry;

import java.util.List;
import jakarta.ejb.Stateless;
import jakarta.inject.Inject;
import com.nilesh.jawarkar.learn.javaee8.control.CarFactory;
import com.nilesh.jawarkar.learn.javaee8.control.CarRepository;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

@Stateless
public class CarManufacturer {
	@Inject
	CarFactory carFactory;
	@Inject
	CarRepository carRepository;
	
	public Car createCar(final Specification spec) throws InvalidEngine {
		if (spec.getEngineType() == EngineType.UNKNOWN)
			throw new InvalidEngine();
		final Car car = carFactory.createCar(spec);
		carRepository.save(car);
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

**CarFactory** -

- It just create Car instance.
- I think dependant or request scope is best suited in this case.
- Lets make it dependant scope (which is any way default scope), it mean its scope is depend on its user bean (its scope depends on the bean  in which it is injected).

``` java
package com.nnj.learn.javaee8.control;

import java.util.UUID;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;
import jakarta.enterprise.context.Dependent;

@Dependent
public class CarFactory {
	public Car createCar(Specification spec) {
		Car c =  new Car();
		c.setColor(spec.getColor());
		c.setEngine(spec.getEngineType());
		c.setIdentifier(UUID.randomUUID().toString());
		return c;
	}
}
```



