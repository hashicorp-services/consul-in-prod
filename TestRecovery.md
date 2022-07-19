TestRecovery.md

## Test Recovery by Simulating Failure of a Node in a Multi-server Cluster

Here we simulate the failure of a single node via various signals and its recovery, and its impact on the cluster state.

These examples assume the following config.json file in each of the nodes (5 nodes in this example)

```
{
"advertise_addr": "192.168.1.11",
"datacenter": "dc1",
"data_dir": "/var/consul",
"encrypt": "EXz7LFN8hpQ4id8EDYiFoQ==",
"node_name": "ConsulServer1",
"server": true,
"bootstrap_expect": 3,
"leave_on_terminate": true,
"skip_leave_on_interrupt": false,
"rejoin_after_leave": true,
"retry_join": [
    "192.168.1.11:8301",
    "192.168.1.12:8301",
    "192.168.1.13:8301"
 ]
}
```

1. Verify that we have a quorum and the Leader is present:

```
root@server1:~#curl -v http://127.0.0.1:8500/v1/status/leader
* Trying 127.0.0.1…
* Connected to 127.0.0.1 (127.0.0.1) port 8500 (#0)
> GET /v1/status/leader HTTP/1.1
> Host: 127.0.0.1:8500
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Type: application/json
< Date: Mon, 24 May 2016 11:08:22 GMT
< Content-Length: 19
< * Connection #0 to host 127.0.0.1 left intact

"192.168.1.11:8300"root@server1:~#
```

1. Also verify that nodes see each other - by checking the Serf peers list

```
root@server1:~# <strong>consul members -detailed</strong>
Node	Address	Status	Tags
ConsulServer1	192.168.1.11:8301	alive	build=0.6.4:26a0ef8c,dc=dc1,expect=3,port=8300,role=consul,vsn=2,vsn_max=3,vsn_min=1
ConsulServer2	192.168.1.12:8301	alive	build=0.6.4:26a0ef8c,dc=dc1,expect=3,port=8300,role=consul,vsn=2,vsn_max=3,vsn_min=1
ConsulServer3	192.168.1.13:8301	alive	build=0.6.4:26a0ef8c,dc=dc1,expect=3,port=8300,role=consul,vsn=2,vsn_max=3,vsn_min=1
```

1. Verify that the Raft peers contains the right set of nodes across every Consul server that should form the quorum, i.e. no left, failed or unreachable servers are present.

```
root@server1:~# cat /var/consul/raft/peers.json
["192.168.1.12:8300","192.168.1.11:8300","192.168.1.13:8300"]
```

1. Ctrl+c to the running Consul SIGINT: Now let us shut down any of the node. I am shutting down the existing leader, in my case it is 192.168.1.11 (server1.example.com).
Server1.example.com

```
==> Caught signal: interrupt
==> Gracefully shutting down agent…
[INFO] consul: server     starting leave
[INFO] raft: Removed peer 192.168.1.12:8300, stopping replication (Index: 739)
[INFO] raft: Removed peer 192.168.1.13:8300, stopping replication (Index: 739)
[INFO] raft: Removed ourself, transitioning to follower
[INFO] raft: Node at 192.168.1.11:8300 [Follower] entering Follower state
[INFO] consul: cluster leadership lost
[INFO] raft: aborting pipeline replication to peer 192.168.1.12:8300
[INFO] raft: aborting pipeline replication to peer 192.168.1.13:8300
[INFO] serf: EventMemberLeave: ConsulServer1.dc1 192.168.1.11
[INFO] consul: removing WAN server ConsulServer1.dc1 (Addr: 192.168.1.11:8300) (DC: dc1)
[INFO] serf: EventMemberLeave: ConsulServer1 192.168.1.11
[INFO] consul: removing LAN server ConsulServer1 (Addr: 192.168.1.11:8300) (DC: dc1)
[INFO] agent: requesting shutdown
[INFO] consul: shutting down server
[INFO] agent: shutdown complete 
```

Now the peers.json file will become null in server1.example.com

```
root@server1:~# cat /var/consul/raft/peers.json
null
```

192.168.1.11 should be deregistered from server2.example.com and server3.example.com root@server2:~# cat /var/consul/raft/peers.json ["192.168.1.12:8300","192.168.1.13:8300"]

```
root@server3:~# cat /var/consul/raft/peers.json
["192.168.1.12:8300","192.168.1.13:8300"]
```

Now the new leader will be elected from the running server2.example.com(192.168.1.12) & server3.example.com(192.168.1.13)

In my case server2.example.com(192.168.1.12) is elected as the new leader.

```
[INFO] consul: removing LAN server ConsulServer1 (Addr: 192.168.1.11:8300) (DC: dc1)
[ERR] agent: failed to sync remote state: No cluster leader
[WARN] raft: Heartbeat timeout reached, starting election
[INFO] raft: Node at 192.168.1.12:8300 [Candidate] entering Candidate state
[INFO] raft: Duplicate RequestVote for same term: 36
[WARN] raft: Election timeout reached, restarting election
[INFO] raft: Node at 192.168.1.12:8300 [Candidate] entering Candidate state
[INFO] raft: Election won. Tally: 2
[INFO] raft: Node at 192.168.1.12:8300 [Leader] entering Leader state
[INFO] consul: cluster leadership acquired
[INFO] consul: New leader elected: ConsulServer2
[INFO] raft: pipelining replication to peer 192.168.1.13:8300
[INFO] consul: member 'ConsulServer1' left, deregistering
```

1. Let’s verify the Consul members

```
root@server2:~# consul members
Node	Address	Status	Type	Build	Protocol	DC
ConsulServer1	192.168.1.11:8301	left	server	0.6.4	2	dc1
ConsulServer2	192.168.1.12:8301	alive	server	0.6.4	2	dc1
ConsulServer3	192.168.1.13:8301	alive	server	0.6.4	2	dc1
```

If the node which left the cluster can be brought up again, we can just restart it and have it rejoin the cluster, returning the cluster to a fully healthy state.

```
root@server1:~# consul agent -config-dir /etc/consul.d/config
```

If any of the node's managed to perform a graceful leave, you may need to have then rejoin the cluster using the join command (whether they restarted or not).

```
root@server1:~# consul join <Node Address>
Successfully joined cluster by contacting 1 nodes.
```

It should be noted that any existing member can be used to issue the join command as the gossip protocol will take care of discovering the server nodes. At this point, the quorum is healthy.

In our case with the above config.json we can simply start the server1.example.com and it will rejoin the cluster because "rejoin_after_leave" was set to true and retry_join keys were provided.

Now we have a quorum with leader and 2 followers.

```
root@server2:~# consul members
Node	Address	Status	Type	Build	Protocol	DC
ConsulServer1	192.168.1.11:8301	alive	server	0.6.4	2	dc1
ConsulServer2	192.168.1.12:8301	alive	server	0.6.4	2	dc1
ConsulServer3	192.168.1.13:8301	alive	server	0.6.4	2	dc1
```

SIGTERM: Now let us kill any of the node. We choose the existing leader (kill )

```
root@server1:~# curl http://127.0.0.1:8500/v1/status/leader
"192.168.1.12:8300"
```

In our case the leader is 192.168.1.12(server2.example.com).

The peers.json file in the killed node will become null.

```
root@server2:~# cat /var/consul/raft/peers.json
null
```

A new leader is elected - ConsulServer1 in this example

```
root@server1:~# curl http://127.0.0.1:8500/v1/status/leader
"192.168.1.11:8300"
```

If the failed server2.example.com is recoverable the easiest option is to bring it back online and have it rejoin the cluster, returning the cluster to a fully healthy state.

Now let’s start Consul:

```
root@server2:~# consul agent -config-dir /etc/consul.d/config
…
[INFO] agent: Joining cluster…
…
[INFO] serf: Re-joined to previously known node: ....
…
[INFO] agent: Synced service 'consul'

```

SIGKILL: In case the node has crashed or sigkill (kill -9 pid) was used, or we are unable to bring up node, we need to rebuild a new Consul server to replace it. Keep in mind that the rebuilt node needs to have the same IP as the failed node because the killed node's IP will be in the other nodes peers.json file. Again, once this node is online, the cluster will return to a fully healthy state provided there should be a leader present that means for bringing up the failed node the other 2 nodes should be in the running state.

If building a new node with the same IP isn't an option, you need to remove the failed node’s IP from raft/peers.json file on all remaining nodes.

Now we will see how the failed/unrecoverable node will be removed from the raft/peers.json file on all remaining nodes.

Stop all remaining nodes. You can attempt a graceful leave or issue a sigterm.

Go to data directory, in this example it is under /var/consul of each Consul node. We need to edit the raft/peers.json file. It should look something like:

```
root@server1:~# cat /var/consul/raft/peers.json
["192.168.1.12:8300","192.168.1.11:8300","192.168.1.13:8300"]
```

Delete the entries for all the failed nodes. We should confirm those servers have indeed failed and will not later rejoin the cluster. We also need to ensure that this file is same across all remaining nodes. It is not enough to just get rid of failed nodes, we also have to make sure that all other healthy nodes are in there. Without it a quorum will not form.

Now if the stopped nodes are successfully stopped then the healthy node's data directory raft/peers.json will be null.

```
root@server1:~# cat /var/consul/raft/peers.json
null
```

We need to remove the failed node’s entry from the existing nodes raft/peers.json file before we add the new node. This is not mandatory but if it’s not done the old nodes will keep attempting to reach the failed one.

In this example, the new node details are server4.example.com with IP 192.168.1.14.

Now let’s start the old nodes with modified raft/peers.json files. After they are started let’s start the new node and join the existing nodes.

Now all the Consul servers peers.json files will look like:

```
root@server1:~# cat /var/consul/raft/peers.json
["192.168.1.14:8300","192.168.1.12:8300","192.168.1.11:8300"]
```

At this point, the cluster should be in an operable state again. One of the nodes should claim leadership and emit the log event:

```
[INFO] consul: cluster leadership acquired
```

We should also verify at this point that one node is the leader and all the others are in the follower state. All the nodes should agree on the peer count as well. This count is (N-1), since a leader does not count itself as a peer.