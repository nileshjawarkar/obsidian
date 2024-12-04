Modify startup.sh/bat and CATALINA_OPTS as follows -

``` sh
export CATALINA_OPTS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"

exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```