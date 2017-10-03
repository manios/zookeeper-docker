Based on commit [9f00dd7](https://github.com/31z4/zookeeper-docker/tree/9f00dd78dcd67baa9b57449329fcbd4744948326) of official Zookeeper image version 3.3.6.

# Supported tags and respective `Dockerfile` links

* `3.4.5`, `latest` [(3.4.5/Dockerfile)](https://github.com/manios/zookeeper-docker/blob/master/3.4.5/Dockerfile)

[![](https://images.microbadger.com/badges/image/manios/zookeeper.svg)](http://microbadger.com/images/manios/zookeeper)  [![build status badge](https://img.shields.io/travis/manios/zookeeper-docker/master.svg)](https://travis-ci.org/manios/zookeeper-docker/branches)

# What is Apache Zookeeper?

Apache ZooKeeper is a software project of the Apache Software Foundation, providing an open source distributed configuration service, synchronization service, and naming registry for large distributed systems. ZooKeeper was a sub-project of Hadoop but is now a top-level project in its own right.

> [wikipedia.org/wiki/Apache_ZooKeeper](https://en.wikipedia.org/wiki/Apache_ZooKeeper)

# How to use this image

## Start a Zookeeper server instance

	$ docker run --name some-zookeeper --restart always -d manios/zookeeper

This image includes `EXPOSE 2181 2888 3888` (the zookeeper client port, follower port, election port respectively), so standard container linking will make it automatically available to the linked containers. Since the Zookeeper "fails fast" it's better to always restart it.

## Connect to Zookeeper from an application in another Docker container

	$ docker run --name some-app --link some-zookeeper:zookeeper -d application-that-uses-zookeeper

## Connect to Zookeeper from the Zookeeper command line client

	$ docker run -it --rm --link some-zookeeper:zookeeper manios/zookeeper zkCli.sh -server zookeeper

## ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml` for `zookeeper`:

```yaml
version: '2'
services:
    zoo1:
        image: manios/zookeeper:3.4.5
        restart: always
        ports:
            - 2181:2181
            - 2888:2888
            - 3888:3888
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2889:3889 server.3=zoo3:2890:3890

    zoo2:
        image: manios/zookeeper:3.4.5
        restart: always
        ports:
            - 2182:2181
            - 2889:2889
            - 3889:3889
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2889:3889 server.3=zoo3:2890:3890

    zoo3:
        image: manios/zookeeper:3.4.5
        restart: always
        ports:
            - 2183:2181
            - 2890:2890
            - 3890:3890
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2889:3889 server.3=zoo3:2890:3890

```

This will start Zookeeper in [replicated mode](http://zookeeper.apache.org/doc/current/zookeeperStarted.html#sc_RunningReplicatedZooKeeper). Run `docker-compose up` and wait for it to initialize completely. Ports `2181-2183` will be exposed.

> Please be aware that setting up multiple servers on a single machine will not create any redundancy. If something were to happen which caused the machine to die, all of the zookeeper servers would be offline. Full redundancy requires that each server have its own machine. It must be a completely separate physical server. Multiple virtual machines on the same physical host are still vulnerable to the complete failure of that host.

Consider using [Docker Swarm](https://www.docker.com/products/docker-swarm) when running Zookeeper in replicated mode.

## Configuration

Zookeeper configuration is located in `/conf`. One way to change it is mounting your config file as a volume:

	$ docker run --name some-zookeeper --restart always -d -v $(pwd)/zoo.cfg:/conf/zoo.cfg manios/zookeeper

## Environment variables

ZooKeeper recommended defaults are used if `zoo.cfg` file is not provided. They can be overridden using the following environment variables.

    $ docker run -e "ZOO_INIT_LIMIT=10" --name some-zookeeper --restart always -d manios/zookeeper

### `ZOO_TICK_TIME`

Defaults to `2000`. ZooKeeper's `tickTime`

> The length of a single tick, which is the basic time unit used by ZooKeeper, as measured in milliseconds. It is used to regulate heartbeats, and timeouts. For example, the minimum session timeout will be two ticks

### `ZOO_INIT_LIMIT`

Defaults to `5`. ZooKeeper's `initLimit`

> Amount of time, in ticks (see tickTime), to allow followers to connect and sync to a leader. Increased this value as needed, if the amount of data managed by ZooKeeper is large.

### `ZOO_SYNC_LIMIT`

Defaults to `2`. ZooKeeper's `syncLimit`

> Amount of time, in ticks (see tickTime), to allow followers to sync with ZooKeeper. If followers fall too far behind a leader, they will be dropped.

### `ZOO_MAX_CLIENT_CNXNS`

Defaults to `60`. ZooKeeper's `maxClientCnxns`

> Limits the number of concurrent connections (at the socket level) that a single client, identified by IP address, may make to a single member of the ZooKeeper ensemble.

## Replicated mode

Environment variables below are mandatory if you want to run Zookeeper in replicated mode.

### `ZOO_MY_ID`

The id must be unique within the ensemble and should have a value between 1 and 255. Do note that this variable will not have any effect if you start the container with a `/data` directory that already contains the `myid` file.

### `ZOO_SERVERS`

This variable allows you to specify a list of machines of the Zookeeper ensemble. Each entry has the form of `server.id=host:port:port`. Entries are separated with space. Do note that this variable will not have any effect if you start the container with a `/conf` directory that already contains the `zoo.cfg` file.

## Where to store data

This image is configured with volumes at `/data` and `/datalog` to hold the Zookeeper in-memory database snapshots and the transaction log of updates to the database, respectively.

> Be careful where you put the transaction log. A dedicated transaction log device is key to consistent good performance. Putting the log on a busy device will adversely affect performance.

# License

View [license information](https://github.com/apache/zookeeper/blob/release-3.4.5/LICENSE.txt) for the software contained in this image.
