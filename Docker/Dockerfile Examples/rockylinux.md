
``` sh
FROM rockylinux:9

RUN yum update -y && yum upgrade -y && \
    yum install -y bind-utils iputils 

CMD ["/bin/bash"]
```
