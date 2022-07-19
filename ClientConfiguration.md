ClientConfiguration.md

## Client Configuration

Consul clients is also a member of a Consul cluster, and can connect to Consul servers for information about the infrastructure.

Consul clients are very light-weight - they simply forward requests as API calls to Consul servers. 

This approach provide a method of insulating Consul servers and 
offloads the responsibility of knowing the servers' addresses from the applications that use Consul.

Create the configuration file under the <tt>/etc/consul.d/client</tt> directory.

```
root@client:~#vim /etc/consul.d/client/config.json
{
 "bind_addr": "192.168.1.21",
 "datacenter": "dc1",
 "data_dir": "/var/consul",
 "encrypt": "EXz7LFN8hpQ4id8EDYiFoQ==",
 "log_level": "INFO",
 "enable_syslog": true,
 "enable_debug": true,
 "node_name": "ConsulClient",
 "server": false,
   "service": {"name": "Apache", "tags": ["HTTP"], "port": 80,
   "check": {"script": "curl localhost >/dev/null 2>&1", "interval": "10s"}},
 "rejoin_after_leave": true,
 "retry_join": [
    "192.168.1.11", 
    "192.168.1.12", 
    "192.168.1.13"
    ]
 }
```

## Start Consul

```
root@client:~#su consul

consul@client:~$consul agent -config-dir /etc/consul.d/client

consul@client:~$consul members
Node	Address	Status	Type	Build	Protocol	DC
ConsulClient	192.168.1.21:8301	alive	client	0.6.4	2	dc1
ConsulServer1	192.168.1.11:8301	alive	server	0.6.4	2	dc1
ConsulServer2	192.168.1.12:8301	alive	server	0.6.4	2	dc1
ConsulServer3	192.168.1.13:8301	alive	server	0.6.4	2	dc1
```

You can see that the client has joined the cluster. Let us try to list the available services from this client using the REST API.

```
consul@client:~$curl -s http://client.example.com:8500/v1/catalog/services
{"ConsulClient": {
   "ID":"ConsulClient",
   "Service":"Apache",
   "Tags":["HTTP"],
   "Address":"",
   "Port":80,
   "EnableTagOverride":false,
   "CreateIndex":0,
   "ModifyIndex":0}
   }
```

We can also list the members via the same REST endpoint.

```
consul@client:~$curl -s http://client.example.com:8500/v1/agent/members

 [{"Name":"ConsulClient",
    "Addr":"192.168.1.21",
    "Port":8301,
    "Tags": {"build":"0.6.4:26a0ef8c",
        "dc":"dc1",
        "role":"node",
        "vsn":"2",
        "vsn_max":"3",
        "vsn_min":"1"},
    "Status":1,
    "ProtocolMin":1,
    "ProtocolMax":3,
    "ProtocolCur":2,
    "DelegateMin":2,
    "DelegateMax":4,
    "DelegateCur":4
   },
 {"Name":"ConsulServer1",
    "Addr":"192.168.1.11",
    "Port":8301,
    "Tags":{"build":"0.6.4:26a0ef8c",
        "dc":"dc1",
        "expect":"3",
        "port":"8300",
        "role":"consul",
        "vsn":"2",
        "vsn_max":"3",
        "vsn_min":"1"},
    "Status":1,
    "ProtocolMin":1,
    "ProtocolMax":3,
    "ProtocolCur":2,
    "DelegateMin":2,
    "DelegateMax":4,
    "DelegateCur":4
   },
 {"Name":"ConsulServer2",
    "Addr":"192.168.1.12",
    "Port":8301,
    "Tags":{"build":"0.6.4:26a0ef8c",
        "dc":"dc1",
        "expect":"3",
        "port":"8300",
        "role":"consul",
        "vsn":"2",
        "vsn_max":"3",
        "vsn_min":"1"},
    "Status":1,
    "ProtocolMin":1,
    "ProtocolMax":3,
    "ProtocolCur":2,
    "DelegateMin":2,
    "DelegateMax":4,
    "DelegateCur":4
   },
  {"Name":"ConsulServer3",
    "Addr":"192.168.1.13",
    "Port":8301,
    "Tags":{"build":"0.6.4:26a0ef8c",
        "dc":"dc1",
        "expect":"3",
        "port":"8300",
        "role":"consul",
        "vsn":"2",
        "vsn_max":"3",
        "vsn_min":"1"},
   "Status":1,
   "ProtocolMin":1,
   "ProtocolMax":3,
   "ProtocolCur":2,
   "DelegateMin":2,
   "DelegateMax":4,
   "DelegateCur":4
   }]
```

In case the client fails or is shutdown, after a restart it will rejoin the cluster automatically, 
as the required parameters are in JSON configuration.