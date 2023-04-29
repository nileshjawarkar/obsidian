
## What make distributed system so appealing : Cons of traditional or centralised systems -
- Vertical scaling - 
- Single point of failure -
- Latency -
- Security and privacy - 

## What is distributed system -
- Process running on different computers
- Communicate with each other over network
- Share state with each other to achieve common goal

### Terminologies
- Node - Process running on dedicated machine
- Cluster - Collection of nodes/computers connected to each other and typically executing the same task (may be same code)

### Zookeeper - High speed coordination service for distributed system
- Distributed system take advantage of parallelism
- To parallelised, system need to allocate the node for execution
- This can be done manually but its not scale-able 
- System need some mechanism(algorithm) which will manage this allocation
- This where co-ordination service like zookeeper comes to picture.
- Zookeeper manage this by electing leader node. Leader node will manage this task.
- But what if leader node goes down. Zookeeper provides a way to re-elect the leader from remaining nodes.
- If previous leader node rejoins the cluster, it will join as normal node.




