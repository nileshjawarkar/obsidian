### Native query in EJB

- Native queries will not work in EJB, as each EJB method already has transaction started. JPA will not allow native query during the active transaction
- For using native query in EJB method, disable the transaction by marking that method with `@Transactional(value = TxType.NOT_SUPPORTED)`

**Example** -
``` java
@Transactional(value = TxType.NOT_SUPPORTED)
public List<Car> retrieveCars(final String filterByAttr, 
							  final String filterByValue) {
	if (filterByAttr == null || "".equals(filterByAttr) 
		|| filterByValue == null) {
		return retrieveCars();
	}
	final List<Car> result = this.entityManager
		.createNativeQuery("select * from Cars where " 
			+ filterByAttr + "= ?1", Car.class)
		.setParameter(1, filterByValue).getResultList();
	return result;
}
```

- I faced this issue in above example and getting exception "ConcurrentModificationException"
- After use of `@Transactional(value = TxType.NOT_SUPPORTED)`, scenario started working.