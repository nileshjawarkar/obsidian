Scope defines the duration within which bean hold its state. In other words scope defines life cycle for the beans.

### CDI scopes
- _@Dependant_ - default. It means bean life-cycle depends on the bean in which it is injected.
- _@RequestScoped_
- _@SessionScoped_
- _@ApplicationScoped_ 

### EJB scopes
- _@Stateless_ bean - Container manages pull of these beans. so we should not add any state to it. We didn't know which bean we will get from pull.
- _@Statefull_ bean - These bean available to user session, so we can add user specific data to it.
- _@Singleton_ bean - This bean is created 1  for application and available for all users. Each method is synchronised.


