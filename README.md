# docker-cassandra
Docker compose to bootstrap Apache Cassandra (2.5.1) cluster with Datastax OpsCenter (5.1)

## Prerequisites

* Docker 18.x
* docker-compose 3.x

```
$ docker -v
Docker version 18.09.0, build 4d60db4

$ docker-compose -v
docker-compose version 1.23.2, build 1110ad01
```

## The Docker Image

The docker image is based off [abh1nav/cassandra](https://registry.hub.docker.com/u/abh1nav/cassandra/)

### Start Cluster

Starting this cluster is as simple as

```
$docker-compose -f docker-compose-cluster.yml up -d
```

The following services are started should start the following:

```
$ docker-compose ps
            Name                  Command      State                        Ports
-------------------------------------------------------------------------------------------------------
docker-cassandra_next_node_1   /sbin/my_init   Up      7000/tcp, 7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp
node00                         /sbin/my_init   Up      7000/tcp, 7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp
nodeops                        /sbin/my_init   Up      0.0.0.0:61620->61620/tcp, 0.0.0.0:8888->8888/tcp
```

### Check status of cluster

```
$ docker exec -it node00 /opt/cassandra/bin/nodetool status

Datacenter: WATERL00
====================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address        Load       Tokens  Owns (effective)  Host ID                               Rack
UN  192.168.240.3  51.34 KB   256     100.0%            615f13f9-0399-42d8-aa81-96a2b0f6eee1  RACK1
UN  192.168.240.4  82.2 KB    256     100.0%            fd52612c-9fac-4617-b841-b12378c633c1  RACK1
```

### Bootstrap a new (next) node to the cluster

```
$ docker-compose up -d --scale next_node=2
node00 is up-to-date
nodeops is up-to-date
Starting docker-cassandra_next_node_1 ... done
Creating docker-cassandra_next_node_2 ... done
```

### Check status of cluster

```
$ docker exec -it node00 /opt/cassandra/bin/nodetool status
Datacenter: WATERL00
====================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address        Load       Tokens  Owns (effective)  Host ID                               Rack
UN  192.168.240.3  51.34 KB   256     100.0%            615f13f9-0399-42d8-aa81-96a2b0f6eee1  RACK1
UN  192.168.240.4  82.2 KB    256     100.0%            fd52612c-9fac-4617-b841-b12378c633c1  RACK1
UJ  192.168.240.5  14.34 KB   256     ?                 288f0c7e-4ef6-462a-9c3c-755daf745965  RACK1
```

### Stop the cluster

```
docker-compose down
```

### Stop and forcibly remove the cluster - use judiciously and cautiously

```
$ docker-compose stop; docker-compose rm -f
Stopping docker-cassandra_next_node_2 ... done
Stopping docker-cassandra_next_node_1 ... done
Stopping node00                       ... done
Stopping nodeops                      ... done
Going to remove docker-cassandra_next_node_2, docker-cassandra_next_node_1, node00, nodeops
Removing docker-cassandra_next_node_2 ... done
Removing docker-cassandra_next_node_1 ... done
Removing node00                       ... done
Removing nodeops                      ... done
```
