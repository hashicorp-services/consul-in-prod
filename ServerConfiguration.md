## Consul Server Configuration

JSON files are edited to configue Consul server files.

Consul has built-in encryption support using a shared-secret system for the Serf Gossip protocol.

In the terminal, we can use the Consul command to generate a key of the necessary length and encoding:

```
root@server1:~#consul keygen
EXz7LFN8hpQ4id8EDYiFoQ==
```

It should be the same for all the servers and clients in a datacenter. 
If it's different the consul members will refuse to join.

Create the first file in the <tt>/etc/consul.d/server</tt> directory for <tt>server1.example.com</tt> (IP Address: 192.168.1.11). Use the encryption key generated above as the value for the "encrypt" key in the JSON.

server1.example.com

```
root@server1:~#vim /etc/consul.d/server/config.json
{
"bind_addr": "192.168.1.11", 
"datacenter": "dc1",
"data_dir": "/var/consul",
"encrypt": "EXz7LFN8hpQ4id8EDYiFoQ==",
"log_level": "INFO",
"enable_syslog": true,
"enable_debug": true,
"node_name": "ConsulServer1",
"server": true,
"bootstrap_expect": 3,
"leave_on_terminate": false,
"skip_leave_on_interrupt": true,
"rejoin_after_leave": true,
"retry_join": [ 
  "192.168.1.11:8301",
  "192.168.1.12:8301",
  "192.168.1.13:8301"
  ]
 }
```

More information on what each key means can be found in the documentation.

You should copy the contents of this configuration file to the other machines that will be acting as your Consul servers. Place them in the same location just like you did in the first server.

The only value you need to modify on the other servers is the IP addresses that it should attempt to listen on - the value of the 'bind_addr' key.

server2.example.com

```
root@server2:~#vim /etc/consul.d/server/config.json
{
"bind_addr": "192.168.1.12",
"datacenter": "dc1",
"data_dir": "/var/consul",
"encrypt": "EXz7LFN8hpQ4id8EDYiFoQ==",
"log_level": "INFO",
"enable_syslog": true,
"enable_debug": true,
"node_name": "ConsulServer2",
"server": true,
"bootstrap_expect": 3,
"leave_on_terminate": false,
"skip_leave_on_interrupt": true,
"rejoin_after_leave": true,
"retry_interval": "30s",
"retry_join": [ 
  "192.168.1.11:8301",
  "192.168.1.12:8301",
  "192.168.1.13:8301"
  ]
 }
```

server3.example.com

```
root@server3:~#vim /etc/consul.d/server/config.json
{
"bind_addr": "192.168.1.13",
"datacenter": "dc1",
"data_dir": "/var/consul",
"encrypt": "EXz7LFN8hpQ4id8EDYiFoQ==",
"log_level": "INFO",
"enable_syslog": true,
"enable_debug": true,
"node_name": "ConsulServer3",
"server": true,
"bootstrap_expect": 3,
"leave_on_terminate": false,
"skip_leave_on_interrupt": true,
"rejoin_after_leave": true,
"retry_interval": "30s",
"retry_join": [ 
  "192.168.1.11:8301",
  "192.168.1.12:8301",
  "192.168.1.13:8301"
  ]
 }
```

Starting the cluster

Now we have everything in place to get a consul cluster up and running.

Start server1.example.com first.

```
root@server1:~#su consul

consul@server1:~$consul agent -config-dir /etc/consul.d/server/

==> WARNING: Expect Mode enabled, expecting 3 servers
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Joining cluster...
    Join completed. Synced with 1 initial agents
==> Consul agent running!
     Node name: 'ConsulServer1'
    Datacenter: 'dc1'
        Server: true (bootstrap: false)
   Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
  Cluster Addr: 192.168.1.11 (LAN: 8301, WAN: 8302)
Gossip encrypt: true, RPC-TLS: false, TLS-Incoming: false
         Atlas: <disabled>
```

The service should start up and occupy the terminal window. It will attempt to connect to the two other servers and keep retrying until they are up.

Now start Consul in the other two servers (server2.example.com, server3.example.com) in the same manner.

server2.example.com

```
root@server2:~#su consul

consul@server2:~$consul agent -config-dir /etc/consul.d/server/

==> WARNING: Expect Mode enabled, expecting 3 servers
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Joining cluster...
    Join completed. Synced with 1 initial agents
==> Consul agent running!
     Node name: 'ConsulServer2'
    Datacenter: 'dc1'
        Server: true (bootstrap: false)
   Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
  Cluster Addr: 192.168.1.12 (LAN: 8301, WAN: 8302)
Gossip encrypt: true, RPC-TLS: false, TLS-Incoming: false
         Atlas: <disabled>
```

server3.example.com

```
root@server3:~#su consul

consul@server3:~$consul agent -config-dir /etc/consul.d/server/

==> WARNING: Expect Mode enabled, expecting 3 servers
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Joining cluster...
    Join completed. Synced with 1 initial agents
==> Consul agent running!
     Node name: 'ConsulServer3'
    Datacenter: 'dc1'
        Server: true (bootstrap: false)
   Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
  Cluster Addr: 192.168.1.13 (LAN: 8301, WAN: 8302)
Gossip encrypt: true, RPC-TLS: false, TLS-Incoming: false
         Atlas: <disabled>
```

These servers (server2.example.com, server3.example.com) will connect to the server1.example.com, 
completing the cluster with one of the servers as the leader and the other two as followers.

List members of the Consul cluster (servers and clients) from any machine:

```
root@server3:~#consul members

Node	Address	Status	Type	Build	Protocol	DC
ConsulServer1	192.168.1.11:8301	alive	server	0.6.4	2	dc1
ConsulServer2	192.168.1.12:8301	alive	server	0.6.4	2	dc1
ConsulServer3	192.168.1.13:8301	alive	server	0.6.4	2	dc1
```

Applications should now connect.

The cluster should now be fully operational.

We'll configure and start the client next.