<figure><a href="https://blog.alexseifert.com/wp-content/uploads/2016/11/Redis_Logo.svg"><img decoding="async" src="Redis_Logo.svg" alt="Redis"></a></figure>

*Note: This article was updated on 9 July 2020 for Redis 6. The previous version was based on Redis 3.2.5. The old repository can still be found as a tag [here](https://github.com/eiskalteschatten/Redis-Sentinel-Docker-Compose/tree/2016).*

[Redis](http://redis.io/) is an easy-to-use solution for anyone looking for a robust key-value store. It is feature-rich, but relatively simple to use and even has [official Docker images](https://hub.docker.com/_/redis/). This post will not go into anymore detail as to what exactly Redis is as it assumes the reader already knows. If not, you can read about it on the [official Redis website](http://redis.io/).

What we will discuss, however, is how to create a failover solution using Redis Sentinel and Docker Compose. There are several code examples in this post, so it might be easier to follow them as well as to understand the project structure [on GitHub](https://github.com/eiskalteschatten/Redis-Sentinel-Docker-Compose).

[Redis Sentinel](http://redis.io/topics/sentinel) is essentially a mode in which the Redis server is started that watches the master Redis instance and chooses a replacement from the slave instances in the event that the master instance is unreachable.

In order for it to do this, the following needs to be configured:

-   We need to define a master instance.
-   We need to setup one or more slave instances.
-   We need to start at least three Sentinel instances.
-   They all need to communicate with each other.

So how do we do all of this?

Redis makes it relatively easy. First, we start a normal instance of Redis which will be the master instance. Then we start additional instances that will become the slaves, but when doing it, we pass a flag with the IP address/hostname and port of the master instance. This flag defines the slaves as slaves and also tells them which instance is the master. An example command looks like this:

```shell
redis-server --slaveof 127.0.0.1 6379
```

Now we have the first two bullet points in our list taken care of, but still need to start at least two Sentinel instances. Redis needs at least two instances so that the Sentinels can “vote” for a slave instance to become master. This is a bit trickier as we first have to define a configuration file. More on that later, but for now, here is the command to use when starting a Sentinel instance:

```shell
redis-server /redis/sentinel.conf --sentinel
```

All of these instances will need to be on separate servers or setup with different configurations if running on the same server. That is beyond the scope of this article though as we are going to isolate each instance in a Docker container as a solution to this problem.

To use Docker, we will need to start multiple containers at once. The best way to do that is with [Docker Compose](https://docs.docker.com/compose/). This article will assume some background knowledge of both Docker and Docker Compose. First we need to create a Compose File that looks something like the following:

**docker-compose.yml:**

```yaml
version: '3.7'

services:

  app:
    image: some-image
    links:
      - redis-sentinel

  redis-master:
    image: redis:6-alpine
    volumes:
      - "./.data:/data"
    ports:
      - "6379:6379"

  redis-slave:
    image: redis:6-alpine
    command: redis-server --slaveof redis-master 6379
    links:
      - redis-master
    volumes:
      - "./.data:/data"

  # Instance 1
  redis-sentinel:
    build:
      context: ./redis-sentinel
    links:
      - redis-master

  # Instance 2
  redis-sentinel2:
    build:
      context: ./redis-sentinel
    links:
      - redis-master

  # Instance 3
  redis-sentinel3:
    build:
      context: ./redis-sentinel
    links:
      - redis-master
```

Here you can see that I have six containers. One for the app where our application will be, one for the Redis master instance, one for a single Redis slave instance and three that serve as Sentinel instances. Alternatively, you can have a single Sentinel container and use the following to scale to multiple instances:

```shell
docker-compose scale redis-sentinel=3
```

It is important that our app communicate with one of the Sentinel instances rather than with one of the Redis instances because otherwise it will have no way of knowing if the master has gone down and which slave has become the master.

I use the official Redis images directly from Docker Hub for the Redis master and slave containers since there is nothing that needs to be changed or configured. Unfortunately since Redis Sentinel requires a configuration file, we will need to create our own Docker image. In the root of the project, I created a folder called “redis-sentinel”. In this folder are three files:

-   Dockerfile
-   sentinel-entrypoint.sh
-   sentinel.conf

“Dockerfile” defines our custom image, the “sentinel-entrypoint.sh” script sets the values in our configuration file and “sentinel.conf” provides the template for the Sentinel configuration.

**redis-sentinel Dockerfile:**

```docker
FROM redis:6-alpine

ENV SENTINEL_QUORUM 2
ENV SENTINEL_DOWN_AFTER 1000
ENV SENTINEL_FAILOVER 1000

RUN mkdir -p /redis

WORKDIR /redis

COPY sentinel.conf .
COPY sentinel-entrypoint.sh /usr/local/bin/

RUN chown redis:redis /redis/* &amp;&amp; \
    chmod +x /usr/local/bin/sentinel-entrypoint.sh

EXPOSE 26379

ENTRYPOINT ["sentinel-entrypoint.sh"]
```

Essentially, the Dockerfile pulls the official Redis image as its base image, sets a few defaults for the Sentinel configuration as environment variables, creates a directory for the other files, then copies them into the container. Port 26379 is also exposed which is the default port for Redis Sentinel. Lastly, we define the “sentinel-entrypoint.sh” script as our entrypoint.

**sentinel-entrypoint.sh:**

```shell
#!/bin/sh

sed -i "s/$SENTINEL_QUORUM/$SENTINEL_QUORUM/g" /redis/sentinel.conf
sed -i "s/$SENTINEL_DOWN_AFTER/$SENTINEL_DOWN_AFTER/g" /redis/sentinel.conf
sed -i "s/$SENTINEL_FAILOVER/$SENTINEL_FAILOVER/g" /redis/sentinel.conf

redis-server /redis/sentinel.conf --sentinel
```

The entrypoint script does nothing other than replace a few strings with the defaults that we defined as environment variables in the Dockerfile, then starts the Redis server using the “–sentinel” flag while pointing it to our configuration file.

**sentinel.conf:**

```shell
port 26379

dir /tmp

sentinel monitor redismaster redis-master 6379 $SENTINEL_QUORUM
sentinel down-after-milliseconds redismaster $SENTINEL_DOWN_AFTER
sentinel parallel-syncs redismaster 1
sentinel failover-timeout redismaster $SENTINEL_FAILOVER
```

The configuration template has the bare-minimum configuration. Here we tell Redis Sentinel which Redis instance is our master (the “redis-master 6379” part of the “sentinel monitor” line) and set a few other settings. “redis-master” is the name of our master image as defined in the Docker Compose file. There are several other things that could be configured, but we will not go into detail here. You can find more detailed information about configuring Redis Sentinel in the [official documentation](http://redis.io/topics/sentinel).

All we have left to do is start Docker Compose with:

```shell
docker-compose up
```

Once all of the containers have started, we can test Sentinel by getting the image id of our “redis-master” image and pausing it so that it “goes down”:

```shell
docker ps
# Copy the image id
docker pause IMAGEID
```

The Sentinel instances should automatically detect that the master is missing then choose a slave to become its replacement while our application should never notice that the master was gone. To restore the master, you can “unpause” it:

```shell
docker unpause IMAGEID
```

As when the master was gone, the Sentinel instances should automatically detect that the master instance is reachable again.

If you have any questions, comments or suggestions, please feel free to leave them in the comments below.

See the full [example project on GitHub](https://github.com/eiskalteschatten/Redis-Sentinel-Docker-Compose).