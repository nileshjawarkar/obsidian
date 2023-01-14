
1) Get
``` sh
curl <URL> -i
OR
curl <URL> -i -XGET
```

- Example
``` sh
curl http://localhost:8080/carman/resources/cars -i 
```

2) Post
``` sh
curl <URL> -i -XPOST -H 'content-type: Application/json' -d '<JSON DATA>'
```

- Example
``` sh
curl http://localhost:8080/carman/resources/cars -i -XPOST -H 'content-type: Application/json' -d '{"engineType":"PETROL"}'
```