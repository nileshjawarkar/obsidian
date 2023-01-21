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

### Injection

The _@Inject_ annotation is CDI's actual workhorse. It allows us to define injection points in the client classes.

### Managing ambiguity

When more than one implementations are available for a type and CDI didn't know which implementation to inject. This is  a ambiguous situation. Solving this ambiguity is easy. CDI, by default, annotates all the implementations of an interface with the _@Default_ annotation. So, we should explicitly tell it which implementation should be injected into the client class my annotating one which you want to be injected by default with _@Default_ and others by annotating with _@Alternative_.

### Life cycle callbacks

- *@PostConstruct* - Method marked with this annotation is called after instance creation
- *@PreDestroy* - Method marked with this annotation is called before destroying the instance


