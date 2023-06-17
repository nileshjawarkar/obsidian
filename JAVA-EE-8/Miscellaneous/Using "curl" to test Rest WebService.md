

1) Get
``` sh
curl <URL> -i
OR
curl <URL> -i -XGET
```

- Example
``` sh
curl http://localhost:8080/carman/resources/cars -i | json_pp
```

2) Post
``` sh
curl <URL> -i -XPOST -H 'content-type: Application/json' -d '<JSON DATA>' | json_pp
```

- Example
``` sh
curl http://localhost:8080/carman/resources/cars -i -XPOST -H 'content-type: Application/json' -d '{"engineType":"PETROL"}' | json_pp
```

- Example
``` shell
curl http://localhost:8080/carman/resources/v2/cars/aa -i -XPOST -H 'content-type: Application/json' -d '["550d776a-e1df-4cc3-955f-c244a30abc06", "0f828a80-6b35-4dc0-b583-46f6c9a30200"]' | json_pp
```