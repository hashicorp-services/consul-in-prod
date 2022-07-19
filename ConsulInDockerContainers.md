
## Deploying Consul in Docker Containers

The first thing we will do is start our Consul cluster. We are going to use a 3 node cluster in the subnet range of 172.16.20.0.

We'll run the following on the first node (Node1):

```
$export HOST_IP=$(ifconfig eth0 | grep 'inet ' |  cut -d: -f2 | awk '{ print $1 }')

$export DOCKER_BRIDGE_IP=$(ifconfig docker0 | grep 'inet ' |  cut -d: -f2 | awk '{ print $1 }')

$docker run -d -h $HOSTNAME -v /mnt:/data \
-p $HOST_IP:8300:8300 \
-p $HOST_IP:8301:8301 \
-p $HOST_IP:8301:8301/udp \
-p $HOST_IP:8302:8302 \
-p $HOST_IP:8302:8302/udp \
-p $HOST_IP:8400:8400 \
-p $HOST_IP:8500:8500 \
-p $DOCKER_BRIDGE_IP:53:53/udp \
progrium/consul -server -advertise $HOST_IP -bootstrap-expect 3
```

As part of this docker run command, we are binding all Consul’s internal ports to the private IP address of our first host, except for the DNS port (53) which is exposed only on the docker0 interface (172.17.0.1 by default). The reason we use the docker bridge interface for the DNS server is that we want all the containers running on the same host to query this DNS interface, but we don’t want anyone from outside doing the same. Since each host will be running a Consul agent, each container will query its own host. We also added the -advertise flag to tell Consul that it should use the host’s IP instead of the docker container’s IP.

On the second host (Node2), we'll run the the same thing but with the additional parameter -join pointing to the first node’s IP:

```
$export HOST_IP=$(ifconfig eth0 | grep 'inet ' |  cut -d: -f2 | awk '{ print $1 }')

$export DOCKER_BRIDGE_IP=$(ifconfig docker0 | grep 'inet ' |  cut -d: -f2 | awk '{ print $1 }')

$docker run -d -h $HOSTNAME -v /mnt:/data \
-p $HOST_IP:8300:8300 \
-p $HOST_IP:8301:8301 \
-p $HOST_IP:8301:8301/udp \
-p $HOST_IP:8302:8302 \
-p $HOST_IP:8302:8302/udp \
-p $HOST_IP:8400:8400 \
-p $HOST_IP:8500:8500 \
-p $DOCKER_BRIDGE_IP:53:53/udp \
progrium/consul -server -advertise $HOST_IP -join 172.16.20.132
Note: 172.16.20.132 is the first node's IP address.
```

We'll do the same for the third node (Node3):

```
$export HOST_IP=$(ifconfig eth0 | grep 'inet ' |  cut -d: -f2 | awk '{ print $1 }')

$export DOCKER_BRIDGE_IP=$(ifconfig docker0 | grep 'inet ' |  cut -d: -f2 | awk '{ print $1 }')

$docker run -d -h $HOSTNAME -v /mnt:/data \
-p $HOST_IP:8300:8300 \
-p $HOST_IP:8301:8301 \
-p $HOST_IP:8301:8301/udp \
-p $HOST_IP:8302:8302 \
-p $HOST_IP:8302:8302/udp \
-p $HOST_IP:8400:8400 \
-p $HOST_IP:8500:8500 \
-p $DOCKER_BRIDGE_IP:53:53/udp \
progrium/consul -server -advertise $HOST_IP -join 172.16.20.132
```
