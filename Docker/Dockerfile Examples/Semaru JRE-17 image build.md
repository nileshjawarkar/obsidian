``` Dockefile
# Stage 1: The "Extract" stage
FROM docker.io/library/debian:stable-slim AS stage01

# Create a temporary directory and copy the local tar file
ENV JRE_FILE=ibm-semeru-open-jre_x64_linux_21.0.10.1.tar.gz 
COPY ./${JRE_FILE} /tmp
# Create the final destination and unzip it here
RUN mkdir -p /opt/ibm/java && \
    tar -xzf /tmp/${JRE_FILE} -C /opt/ibm/java --strip-components=1

# Stage 2
FROM docker.io/library/debian:stable-slim

# ONLY copy the extracted folder from the first stage
COPY --from=stage01 /opt/ibm/java /opt/ibm/java

# Set environment variables for the version and installation path
ENV JAVA_VERSION=21.0.10.1 \
    JAVA_HOME=/opt/ibm/java \
    PATH="/opt/ibm/java/bin:${PATH}"

# Install dependencies, download Semeru, and clean up
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Verify installation
RUN java -version

CMD ["jshell"]
```