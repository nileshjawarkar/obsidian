
- ### We are going to use Car manufacturing example to  discuss various concepts in coming sections

	1) Create a Car as per provided specifications such as color and engine
		- Save the car
	2) Retrieve all the cars
	3) Retrieve all the car matching provided attribute
	4) Retrieve car with selected id

- ### Above description indicates that we will require following classes

	 
	 1) Color - Enum
	 2) EngineType - Enum
	 3) Specification 
	 4) Car
	 5) CarRepository - To save the car to DB to something else
	 6) CarManufacturer - Manage create, retrieval


- ### Example Implementation of above classes 

	*Note* - Please create maven project as explained in [section](obsidian://open?vault=obsidian&file=JAVA-EE-8%2FMiscellaneous%2FSetting%20JAVA%20EE%20maven%20project%20in%20eclipse)
	

1) Color Enum
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

public enum Color {
	RED, WHITE, BLUE
}
```
2) EngineType Enum
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

public enum EngineType {
	PETROL, DIESEL, ELECTRIC, UNKNOWN
}

//-- UNKNOWN - Added for demo purpose - Domo of exception mapper.
//-- If we get this as engine type CarManufacturer call will throw InvalidEngineException.
//-- Now to manage this exception in rest-webservice, we will implement Exception mapper.
```
3) Specification class
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

import javax.validation.constraints.NotNull;

public class Specification {
	Color color;
	EngineType engineType;
	
	public Color getColor() {
		return color;
	}
	
	public EngineType getEngineType() {
		return engineType;
	}
	
	public void setColor(Color color) {
		this.color = color;
	}
	
	public void setEngineType(EngineType engineType) {
		this.engineType = engineType;
	}
}
```
4) Car class
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

public class Car {
	private String             id;
	private EngineType         engineType;
	private Color              color;

	public Color getColor() {
		return color;
	}

	public EngineType getEngineType() {
		return engineType;
	}

	public String getId() {
		return id;
	}

	public void setColor(final Color color) {
		this.color = color;
	}

	public void setEngine(final EngineType engineType) {
		this.engineType = engineType;
	}

	public void setEngineType(final EngineType engineType) {
		this.engineType = engineType;
	}

	public void setId(final String id) {
		this.id = id;
	}

	public void setIdentifier(final String id) {
		this.id = id;
	}

}

```

5) CarRepository interface
``` java
package com.nilesh.jawarkar.javaee8.control;

import java.util.List;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;

public interface CarRepository {
	public Car findById(final String id) ;
	public List<Car> getAll(final String filterByAttr, final String filterByValue) ;
	public void save(final Car car);
}
```

6) CarRepositoryInMemImpl class
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;

public class CarRepositoryInMemImpl implements CarRepository {

	private List<Car> list = new ArrayList<>();

	public CarRepository() {
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
	public List<Car> getAll(final String filterByAttr, 
		final String filterByValue) {
		if (filterByAttr == null 
			|| filterByAttr.equals("") 
			|| filterByValue == null
			|| filterByValue.equals(""))
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

7) CarFactory class
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

public class CarFactory {
	public Car createCar(Specification spec) {
		Car c = new Car();
		c.setColor(spec.getColor());
		c.setEngine(spec.getEngineType());
		return c;
	}
}
```

8) InvalidEngine class
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

public class InvalidEngine extends RuntimeException {
	private static final long serialVersionUID = 1L;
	public InvalidEngine(String message) {
		super(message);
	}
}
```

9) CarManufacturer class
``` java
package com.nilesh.jawarkar.learn.javaee8.boundry;

import java.util.List;
import com.nilesh.jawarkar.learn.javaee8.control.CarFactory;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.EngineType;
import com.nilesh.jawarkar.learn.javaee8.entity.InvalidEngine;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;
import com.nilesh.jawarkar.learn.javaee8.entity.InvalidEngine;

public class CarManufacturer {
	CarFactory        carFactory;
	CarRepository carRepository;

	public CarManufacturer(CarFactory carFactory, 
		CarRepository carRepository) {
		this.carFactory = carFactory;
		this.carRepository = carRepository;
	}

	public Car createCar(final Specification spec) {
		if (spec.getEngineType() == null 
		|| spec.getEngineType() == EngineType.UNKNOWN)
			throw new InvalidEngine("Engine type not supported.");
			
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
		return carRepository.getAll(filterByAttr, filterByValue);
	}
}
```

