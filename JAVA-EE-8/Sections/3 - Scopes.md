*CDI scopes*
- dependant - default. It means bean life-cycle depends on the bean in which it is injected.
- session scope
- application scope 

*EJB scopes*
- stateless bean - Container manages pull of these beans. so we should not add any state to it. We didn't know which bean we will get from pull.
- statefull bean - These bean available to user session, so we can add user specific data to it.
- singleton bean - This bean is created 1  for application and available for all users. Each method is synchronised.

