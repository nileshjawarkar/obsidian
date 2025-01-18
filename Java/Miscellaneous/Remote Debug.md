
- Compile code in debug mode -
``` sh
javac -g OurApplication.java
```

- Start app with jdwp (Java Debug Wire Protocol)
```sh

# Example 1
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=127.0.0.1:8000 OurApplication 

# Example 2
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,\
address=127.0.0.1:8000 \
co.in.nnj.learn.executors.Ex1_SingleThreadExecutor
```

- IDE  (eclipse,idea,vscode,neovim) - Attach to remote port 8000

Link -

1) https://www.baeldung.com/java-application-remote-debugging