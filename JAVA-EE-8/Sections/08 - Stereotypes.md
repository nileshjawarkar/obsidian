A **stereotype** is a kind of annotation, applied to a bean, that incorporates other annotations. Stereotypes can be particularly useful in large applications where you have a number of beans that perform similar functions. A stereotype is a kind of annotation that specifies the following:

-   A default scope
-   Zero or more interceptor bindings
-   Optionally, a @Named annotation, guaranteeing default EL naming
-   Optionally, an @Alternative annotation, specifying that all beans with this stereotype are alternatives

A bean annotated with a particular stereotype will always use the specified annotations, so you do not have to apply the same annotations to many beans.

For example, you might create a stereotype named Action, using the "javax.enterprise.inject.Stereotype" annotation:

``` java
@Stereotype
@RequestScoped
@Secure
@Transactional
@Named
@Target(TYPE)
@Retention(RUNTIME)
public @interface Action {}
```
