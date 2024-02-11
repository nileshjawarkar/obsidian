**Producer** - They're also the easiest way to integrate objects which are not beans into the CDI environment. 
- According to the specs , a producer method acts as a source of objects to be injected, where:
	- the objects to be injected are not required to be instances of beans,
	- the concrete type of the objects to be injected may vary at runtime or
	- the objects require some custom initialisation that is not performed by the bean constructor
- @Produces - Annotation can used to decorate the method, filed to mark them as producer.

In "CarResourceV2", we are using some Color as defaultColor. But Color is Enum not a bean. In such cases, we can use producers. Here we created a class with method which is marked with Produces annotation.

``` java
package com.nnj.learn.javaee8.control;

import com.nnj.learn.javaee8.entity.Color;
import javax.enterprise.inject.Produces;

public class DefaultColorProducer {
	@Produces
	public Color getDefaultColor() {
		return Color.RED;
	}
}
```

Now we can modify "CarResourceV2" class to inject it.

``` java
@Inject
Color defaultColor;
//-- Color defaultColor = Color.RED;
```

But in this case, we have to generated one default value, what if we need to generate multiple. In such cases we need to combine it with qualifiers. It can be name qualifier provided by JavaEE or we can create one custom qualifier.

**Named qualifier** - We can combine Named qualifier with producer to generate required value. Lets modify the example we seen earlier to produce multiple values but qualified with named qualifier .

``` java
package com.nnj.learn.javaee8.control;

import com.nnj.learn.javaee8.entity.Color;
import javax.enterprise.inject.Produces;
import javax.inject.Named;

public class DefaultColorProducer {
	@Produces
	@Named("Commom")
	public Color getDefaultColor() {
		return Color.RED;
	}
	
	@Produces
	@Named("SpeatialEdition")
	public Color getSpeatialEditionColor() {
		return Color.BLUE;
	}
}
```

Now "DefaultColorProducer" generate 2 values, due to this we need to modify "CarResourceV2" to inject the default color.

``` java
@Inject
@Named("SpeatialEdition")
Color defaultColor;
```

We qualified the value to be injected using named qualifier. If we dont do it, web-app container will complain and deployment will fail.

**Custom Qualifier** -
We can do the same thing using custom/user defined qualifier. Lets see how to do it.
Define custom qualifier -
``` java
package com.nnj.learn.javaee8.control;

import java.lang.annotation.Documented
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import javax.inject.Qualifier;

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface SpeatialEditionColor {
}
```

Use it with producer and at the injection point -

``` java
package com.nnj.learn.javaee8.control;

import com.nnj.learn.javaee8.entity.Color;
import javax.enterprise.inject.Produces;
import javax.inject.Named;

public class DefaultColorProducer {
	@Produces
	@Named("Commom")
	public Color getDefaultColor() {
		return Color.RED;
	}
	
	@Produces
	//-- @Named("SpeatialEdition")
	@SpeatialEditionColor
	public Color getSpeatialEditionColor() {
		return Color.BLUE;
	}
}
```

CarResourceV2 -
``` java
@Inject
//@Named("SpeatialEdition")
@SpeatialEditionColor
Color defaultColor;
```


