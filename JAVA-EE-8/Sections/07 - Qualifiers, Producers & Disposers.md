**Qualifiers** - You can use qualifiers to provide various implementations of a particular bean type. A qualifier is an annotation that you apply to a bean. 
A qualifier type is a Java annotation defined with following annotations
- @Qualifier
- @Target({METHOD, FIELD, PARAMETER, TYPE}) 
- @Retention(RUNTIME).

**Producer** - They're also the easiest way to integrate objects which are not beans into the CDI environment. 
- According to the specs , a producer method acts as a source of objects to be injected, where:
> -   the objects to be injected are not required to be instances of beans,
> -   the concrete type of the objects to be injected may vary at runtime or
> -   the objects require some custom initialisation that is not performed by the bean constructor
- @Produces - Annotation can used to decorate the method, filed to mark them as producer.
**Disposers** - Disposers are used rarely but good to know feature. It will come handy when you want to do custom cleanup on some type. Disposer will be called by the container before destroying target type.
- @Disposes - Parameter of the cleanup method need to be decorated with @Disposes annotation.
- Disposer need to be used with producer.

### Producer using qualifier @Named (Provided by JAVAEE. Not type safe)
- Define producer
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
	
	public void cleanUp(@Disposes Color color) {
		// -- Some custome clean up
		color = null;
	}
}

```

- Use producer
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


### Producer using custom qualifier 
- Define custom qualifier
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

- Define producer
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import javax.enterprise.inject.Disposes;
import javax.enterprise.inject.Produces;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.Diesel;

public class DefaultColorProducer {

	// -- @Named("Red")
	@Diesel
	@Produces
	public Color setDefaultColor() {
		return Color.RED;
	}

	public void cleanUp(@Disposes Color color) {
		// -- Some custome clean up
		color = null;
	}
}

```

- Use custom producer
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
```

