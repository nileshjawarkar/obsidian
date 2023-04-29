How we are going to develop for distributed system, when we didn't have multiple computers. 
- Typically distributed application development done as we do development traditionally, on single computer.

### Download and installation
- Download latest version from [URL](https://zookeeper.apache.org/releases.html)
- Unzip the downloaded file

### Configuration
- Downloaded zip-file has 2 important directories "bin" and "conf".
- Create new directory logs (under root of unzipped directory)
![[Pasted image 20230415160307.png]]

- Now go to "conf" directory and duplicate the file named "zoo_sample.cfg" and rename new file to "zoo.cfg"
- This is the config file zookeeper uses for the configuration.
- Point to note - This file start the zookeeper in standalone mode. That is what we want for the development.
- Now open the file in some editor. File has many properties, just take note of _clientPort=2181_. This defines the port which zookeeper will use. We will go with the defaults for now and will not modify anything.

### Start the server

- Go to the "bin" direcory
- And run the server using command - "./zkServer.sh start"

### CLI - App connect to zookeeper and allow to execute some commands

- Run cli using command - "./zkCli.sh"

Basic CLI commands
- help
- ls PATH
- create PATH "DATA"
- delete PATH

