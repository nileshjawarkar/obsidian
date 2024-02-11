Java applications generally store configuration as key-value strings in the property files. We can inject these config values in the application using CDI. Lets see the example implementation for it...

Consider our application writes config values to "application.properties" file. We will read it and inject it in the bean using custom qualifier "config".

Create custom qualifier "config" -
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import jakarta.enterprise.util.Nonbinding;
import jakarta.inject.Qualifier;

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AppConfig {
    @Nonbinding
	String value();
}
```

Create producer - We can mark this class as "ApplicationScoped", in order to the load application.properties file only once.
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.io.InputStream;
import java.util.Properties;
import jakarta.annotation.PostConstruct;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Produces;
import jakarta.enterprise.inject.spi.InjectionPoint;

@ApplicationScoped
public class ConfigExposer {
	private Properties configProperties = null;
	
	@PostConstruct
	public void init() {
		try( InputStream ios = ConfigExposer.class
				.getResourceAsStream("/application.properties") ) {
			configProperties = new Properties();
			configProperties.load(ios);
		} catch (Exception e) {
		}
	}
	
	@Produces
	@AppConfig("unused")
	public String exposeConfig(InjectionPoint injectionPoint) {
		String key = injectionPoint.getAnnotated()
			.getAnnotation(AppConfig.class).value();
		String value = "";
		if(key != null && !key.equals("")) {
			value = configProperties.getProperty(key);
		}
		return value;
	}
}
```

Example "application.properties" file - This file must be available in class path.
``` Text
car.name_prefix=xyz
```

Using the config qualifier -
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.util.UUID;
import com.nilesh.jawarkar.learn.javaee8.entity.Car;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;
import jakarta.enterprise.context.Dependent;
import jakarta.inject.Inject;

@Dependent
public class CarFactory {

	@Inject
	@AppConfig("car.name_prefix")
	String prefix;
	
	public Car createCar(Specification spec) {
		Car c =  new Car();
		c.setColor(spec.getColor());
		c.setEngine(spec.getEngineType());
		c.setIdentifier(prefix + UUID.randomUUID().toString());
		return c;
	}
}
```