1) Named - Provided by JAVAEE. Not type safe.
- Producer
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import javax.enterprise.inject.Produces;
import javax.inject.Named;

import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.Diesel;

public class DefaultColorProducer {
	
	@Named("Red")
	@Produces
	public Color setDefaultColor() {
		return Color.RED;
	}
}

```

- Consumer
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.UUID;
import javax.inject.Inject;

import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.Diesel;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

public class CarFactory {
	
	@Named("Red")
	@Inject
	Color defaultColor;
	
	public Car createCar(Specification specs) {
		Car car = new Car();
		car.setIdentifier(UUID.randomUUID().toString());
		car.setColor(specs.getColor() == null? defaultColor : specs.getColor());
		car.setEngine(specs.getEngineType());
		return car;
	}
}

```


2) Custom - Created by user. Example - Diesel.
- Qualifier
``` java
package com.nilesh.jawarkar.learn.javaee8.entity;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

import javax.inject.Qualifier;

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Diesel {
}

```
- Producer
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import javax.enterprise.inject.Produces;
import javax.inject.Named;

import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.Diesel;

public class DefaultColorProducer {
	
	@Diesel
	@Produces
	public Color setDefaultColor() {
		return Color.RED;
	}
}

```
- Consumer
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.UUID;
import javax.inject.Inject;

import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.Diesel;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

public class CarFactory {
	
	@Diesel
	@Inject
	Color defaultColor;
	
	public Car createCar(Specification specs) {
		Car car = new Car();
		car.setIdentifier(UUID.randomUUID().toString());
		car.setColor(specs.getColor() == null? defaultColor : specs.getColor());
		car.setEngine(specs.getEngineType());
		return car;
	}
}
