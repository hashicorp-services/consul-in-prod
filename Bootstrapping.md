## Bootstrapping

TODO: Edit content from https://imaginea.gitbooks.io/consul-devops-handbook/content/bootstrapping.html

An agent can run in both client and server mode. Server nodes are responsible for running the consensus protocol and storing the cluster state. The client nodes are mostly stateless and rely heavily on the server nodes.

Before a Consul cluster can begin to service requests, a server node must be elected leader. 
Thus, the first nodes that are started are generally the server nodes.
Bootstrapping is the process of joining these initial server nodes into a cluster.

The recommended way to bootstrap is to use the <tt><strong>-bootstrap-expect</strong></pre> option. 
This option informs Consul of the expected number of server nodes and automatically starts the leader election process when that many servers are available. 

To prevent inconsistencies and split-brain situations (that is, clusters where multiple servers consider themselves leader), 
all servers should either specify the same value for <tt>-bootstrap-expect</tt> or specify no value at all. 
Only servers that specify a value will attempt to bootstrap the cluster.

<a target="_blank" href="https://www.consul.io/docs/internals/consensus.html#deployment_table">HashiCorp recommends</a> 3 or 5 total servers per datacenter. 
A single server deployment is highly discouraged as data loss is inevitable in a failure scenario. 

Here we are considering a 3 server cluster. We can start Node1, Node2, and Node3 with the -bootstrap-expect flag set to 3 in each. 
Once the first node is started, you should see a message like below:

```
[WARN] raft: EnableSingleNode disabled, and no known peers. Aborting election.
```

This indicates that the node is expecting 2 peers but none are known yet. You will see a similar message when you start the second node. To prevent a split-brain scenario, the servers will not elect themselves leader until the number specified in bootstrap-expect comes up.

Once you start the third node, you should see leader election taking place and one of the server nodes will elect itself the leader, you should see a INFO something similar to:

```
[INFO] raft: Election won. Tally: 2
[INFO] raft: Node at 192.168.1.11:8300 [Leader] entering Leader state
[INFO] consul: cluster leadership acquired
[INFO] consul: New leader elected: ConsulServer1
```

## Manual Bootstrapping

In versions of Consul prior to 0.4, bootstrapping was a manual process. For details on using the -bootstrap flag directly, see the manual bootstrapping guide. Manual bootstrapping is not recommended as it is more error-prone than automatic bootstrapping with -bootstrap-expect. It is recommended to use -bootstrap-expect but we mention this for completeness.