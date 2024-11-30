**Producer** - They're also the easiest way to integrate objects which are not beans into the CDI environment. 
- According to the specs , a producer method acts as a source of objects to be injected, where:
	- the objects to be injected are not required to be instances of beans,
	- the concrete type of the objects to be injected may vary at runtime or
	- the objects require some custom initialisation that is not performed by the bean constructor
- @Produces - Annotation can used to decorate the method, filed to mark them as producer.

In "CarResourceV2", we are using some Color RED as default color. We can inject that default color using CDI, but note that Color is Enum not a bean. In such cases, we can use producers to produce those default values. 

Here we created a class with method which is marked with Produces annotation.

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

In this case, we have generated one default value. But what if we need to generate multiple values as shown in case below. We can not do this because it will cause ambiguity for container/server which leads to the deployment failure.

``` java
package com.nnj.learn.javaee8.control;

import com.nnj.learn.javaee8.entity.Color;
import javax.enterprise.inject.Produces;
import javax.inject.Named;

public class DefaultColorProducer {
	@Produces
	public Color getDefaultColor() {
		return Color.RED;
	}
	
	@Produces
	public Color getSpeatialEditionColor() {
		return Color.BLUE;
	}
}
```

In such cases we need to combine producer with qualifier. 
- It can be named qualifier provided by JavaEE or 
- we can create one custom qualifier.

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

Now "DefaultColorProducer" implementation generates 2 values and we can modify "CarResourceV2" to inject the default color as follows -

``` java
@Inject
@Named("SpeatialEdition")
Color defaultColor;
```

One problem with named qualifier is that it is not type safe. To manage this injection in type safe way, we need to define custom qualifier.

**Custom Qualifier** - Lets define a custom qualifier "SpeatialEditionColor" as follows

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


