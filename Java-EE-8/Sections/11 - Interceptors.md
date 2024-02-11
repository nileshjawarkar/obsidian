As name suggest interceptors intercept request. Interceptors is one of the way to execute cross cutting concerns. Cross cutting concerns is the functionality which is not part of business domain. Such as you want track favourite color. Such functionality should be implemented by using interceptors.

- Add decorate method with @Interceptors annotation  
``` java
@Interceptors(TrackFavouriteColor.class)
public Car createCar(final Specification spec) {
	if (spec.getEngineType() == EngineType.UNKNOWN)
		throw new InvalidEngine();
	
	final Car car = carFactory.createCar(spec);
	carRepository.save(car);
	return car;
}
```

- Implement the class which execute on method invocation. This class need to be marked with @Interceptor annotation and provides a method with signature "Object methodName(InovacationContext context)"" and marked with @AroundInvoke annotation. 
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import javax.interceptor.AroundInvoke;
import javax.interceptor.Interceptor;
import javax.interceptor.InvocationContext;

import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

@Interceptor
public class TrackFavouriteColor {

	@AroundInvoke
	Object arroundInvoke(final InvocationContext context) throws Exception {
		final Object[] args = context.getParameters();
		for (final Object arg : args) {
			if (arg instanceof Specification) {
				final Specification spec = (Specification) arg;
				final Color         c    = spec.getColor();
				System.out.println("Asked car of color - " + c.name());
			}
		}
		//-- This is very important and execute the actual method.
		return context.proceed();
	}
}

```

**Binding custom interceptors**

Same thing we done in above example, we will do it now with custom annotation.

- Implement annotation with @InterceptorBinding
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import javax.enterprise.util.Nonbinding;
import javax.inject.Qualifier;
import javax.interceptor.InterceptorBinding;
import com.nilesh.jawarkar.learn.javaee8.entity.Color;

@Qualifier
@Documented
@Target({ ElementType.METHOD, ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
@InterceptorBinding
public @interface TrackColor {
	@Nonbinding
	Color value();
}
```

- Modify the tracker class and decorate the class with @TrackColor annotation
``` java
package com.nilesh.jawarkar.learn.javaee8.control;

import javax.annotation.Priority;
import javax.interceptor.AroundInvoke;
import javax.interceptor.Interceptor;
import javax.interceptor.InvocationContext;

import com.nilesh.jawarkar.learn.javaee8.entity.Color;
import com.nilesh.jawarkar.learn.javaee8.entity.Specification;

@Interceptor
@TrackColor(Color.ANY)
@Priority(Interceptor.Priority.APPLICATION) // This is required to enable the Interceptor
public class TrackFavouriteColor {

	@AroundInvoke
	Object arroundInvoke(final InvocationContext context) throws Exception {
		final TrackColor trackColor =  
		    context.getMethod().getAnnotation(TrackColor.class);
		final Color      color      = trackColor.value();
		final Object[]   args       = context.getParameters();
		for (final Object arg : args) {
			if (arg instanceof Specification) {
				final Specification spec = (Specification) arg;
				final Color         c    = spec.getColor();
				if (color == Color.ANY || color == c) {
					System.out.println("Asked car of color - " + c.name());
				}
			}
		}
		return context.proceed();
	}
}
```

- Apply the @TrackColor annotation to the method
``` java
// -- @Interceptors(TrackFavouriteColor.class)
@TrackColor(Color.ANY)
public Car createCar(final Specification spec) {
	if (spec.getEngineType() == EngineType.UNKNOWN)
		throw new InvalidEngine();
	
	final Car car = carFactory.createCar(spec);
	carRepository.save(car);
	return car;
}
```