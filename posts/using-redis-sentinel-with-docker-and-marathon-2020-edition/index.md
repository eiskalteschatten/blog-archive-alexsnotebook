<figure><img loading="lazy" decoding="async" src="martins-zemlickis-NPFu4GfFZ7E-unsplash.jpg" alt=""></figure>

*Note: This article expands upon the principles as explained in my other post about [Redis Sentinel and Docker Compose](https://blog.alexseifert.com/2020/07/13/using-redis-sentinel-with-docker-compose-2020-edition/). If you haven’t read it yet, you should do so before continuing with this article.*

**Using Redis Sentinel with Docker and Marathon is a relatively complex procedure that requires every instance of Redis to be able to communicate with all other instances.**

Using [Redis Sentinel](http://redis.io/topics/sentinel) with [Docker](https://www.docker.com/) and [Marathon](https://mesosphere.github.io/marathon/) is a relatively complex procedure that requires every instance of Redis to be able to communicate with all other instances. The Sentinels have to talk to both the master and the slaves while the slaves have to be able to synchronize with the master.

Since there will be a lot of code in this article, I have created a [GitHub repository](https://github.com/Developers-Notebook/Redis-Sentinel-Docker-Marathon) where it might be easier to follow and understand.

The example in this post will work with the same setup as defined in my other article about [Redis Sentinel and Docker Compose](https://blog.alexseifert.com/2016/11/14/using-redis-sentinel-with-docker-compose/):

-   We need to define a master instance.
-   We need to setup one or more slave instances.
-   We need to start at least three Sentinel instances.
-   They all need to communicate with each other.

This setup is relatively easy to accomplish with Docker Compose, but what if we want each instance to run in its own Docker container with Marathon? That is where things begin to get a little more complex.

Marathon Configuration
----------------------

First of all, we need to setup our Marathon configuration so that it deploys our Docker images properly. Essentially, this is the same as the [“docker-compose.yml” file from my last post](https://github.com/eiskalteschatten/Redis-Sentinel-Docker-Compose/blob/master/docker-compose.yml), but in JSON format plus a couple of extra necessary parameters for Marathon:

**marathon.json:**

```json
{
  "id": "/",
  "apps": [
    {
      "id": "/app",
      "cpus": 0.01,
      "mem": 32,
      "instances": 1,
      "dependencies": [
        "/redis-sentinel"
      ],
      "container": {
        "type": "DOCKER",
        "docker": {
          "image": "some-image",
          "network": "BRIDGE"
        }
      },
      "healthChecks": [
        {
          "path": "/",
          "portIndex": 0,
          "protocol": "HTTP",
          "gracePeriodSeconds": 30,
          "intervalSeconds": 10,
          "timeoutSeconds": 3,
          "maxConsecutiveFailures": 3,
          "minimumHealthCapacity": 1
        }
      ]
    },
    {
      "id": "/redis-master",
      "cpus": 0.01,
      "mem": 256,
      "instances": 1,
      "labels": {
        "MARATHON_SINGLE_INSTANCE_APP": "true"
      },
      "container": {
        "type": "DOCKER",
        "docker": {
          "image": "redis:6-alpine",
          "network": "BRIDGE",
          "portMappings": [
            {
              "containerPort": 6379,
              "hostPort": 31111
            }
          ]
        },
        "volumes": [
          {
            "containerPath": "/data",
            "hostPath": "/some/path/",
            "mode": "RW"
          }
        ]
      },
      "upgradeStrategy": {
        "minimumHealthCapacity": 0,
        "maximumOverCapacity": 0
      },
      "constraints": [
        [
          "hostname",
          "CLUSTER",
          "node.for.redis.master.com"
        ]
      ],
      "healthChecks": [
        {
          "portIndex": 0,
          "protocol": "TCP",
          "gracePeriodSeconds": 300,
          "intervalSeconds": 60,
          "timeoutSeconds": 20,
          "maxConsecutiveFailures": 1
        }
      ]
    },
    {
      "id": "/redis-slave",
      "cpus": 0.01,
      "mem": 256,
      "instances": 2,
      "dependencies": [
        "/redis-master"
      ],
      "container": {
        "type": "DOCKER",
        "docker": {
          "image": "/path/to/docker/images/redis-slave",
          "network": "BRIDGE",
          "portMappings": [
            {
              "containerPort": 6379,
              "hostPort": 0,
              "servicePort": 6379
            }
          ]
        },
        "volumes": [
          {
            "containerPath": "/data",
            "hostPath": "/some/path/",
            "mode": "RW"
          }
        ]
      },
      "env": {
        "MASTER_HOST": "node.for.redis.master.com",
        "MASTER_PORT": "31111"
      },
      "healthChecks": [
        {
          "portIndex": 0,
          "protocol": "TCP",
          "gracePeriodSeconds": 300,
          "intervalSeconds": 60,
          "timeoutSeconds": 20,
          "maxConsecutiveFailures": 1
        }
      ]
    },
    {
      "id": "/redis-sentinel",
      "cpus": 0.01,
      "mem": 32,
      "instances": 3,
      "dependencies": [
        "/redis-master",
        "/redis-slave"
      ],
      "container": {
        "type": "DOCKER",
        "docker": {
          "image": "/path/to/docker/images/redis-sentinel",
          "network": "BRIDGE",
          "portMappings": [
            {
              "containerPort": 26379,
              "hostPort": 0,
              "servicePort": 26379
            }
          ]
        }
      },
      "env": {
        "MASTER_HOST": "node.for.redis.master.com",
        "MASTER_PORT": "31111"
      },
      "healthChecks": [
        {
          "portIndex": 0,
          "protocol": "TCP",
          "gracePeriodSeconds": 300,
          "intervalSeconds": 60,
          "timeoutSeconds": 20,
          "maxConsecutiveFailures": 1
        }
      ]
    }
  ]
}
```

I won’t go into a lot of detail about configuring Marathon, but I will cover the parts that specifically affect running Redis Sentinel. You can find more information about configuring Marathon to run Docker containers [in the Marathon documentation](https://mesosphere.github.io/marathon/docs/native-docker.html).

As in the “docker-compose.yml” file, we have four containers: one for our app, one for the Redis master, one for the Redis slave and one for Redis Sentinel. For each container, I have set some specific settings which I will now explain.

**App**

-   The “id” I’ve given the app is “/app” which is equivalent to “app” in the docker-compose.yml file.
-   The app only gets 1 instance, but that can be changed to whatever you need.
-   The app has “/redis-sentinel” as a dependency. That means Marathon will not start it until “/redis-sentinel” is running.

**Redis Master**

-   The “id” I’ve given the Redis master is “/redis-master” which is equivalent to “redis-master” in the docker-compose.yml file.
-   The master only gets 1 instance and this is set in stone since we will assign it a specific hostname and port.
-   The master also has a “labels” section as well as a bit further down an “upgradeStrategy” section. The contents of these two ensure that there can only ever be a maximum of one master instance. “MARATHON\_SINGLE\_INSTANCE\_APP” makes it a single instance app and the settings in “upgradeStrategy” prevents Marathon from starting additional instances while deploying.
-   The “hostPort” setting under “portMappings” sets our fixed port for the master. This can be changed to whatever you need, but should stay consistent later when the environment variables are set for the slave and Sentinel containers.
-   The “volumes” section defines a spot on the server where we can save the Redis data for persistence.
-   The “constraints” section is where we define the fixed host for the master instance. If your server is behind a load balancer, for example, this should be a slave. I’ve used the placeholder “node.for.redis.master.com”.

**Redis Slave**

-   The “id” I’ve given the Redis slave is “/redis-slave” which is equivalent to “redis-slave” in the docker-compose.yml file.
-   The slave only gets 2 instances by default, but that can be change to whatever you feel is necessary.
-   The slave has “/redis-master” as a dependency. That means Marathon will not start it until “/redis-master” is running.
-   I’ve set the path to our custom Docker image in the “image” section under “docker”. What value this needs will depend on where you’ve put your built docker images for deployment.
-   The “env” section is where I set the environment variables needed to tell the slave where the master is. The host and port should match the host and port you assigned the master above. More details on this later.

**Redis Sentinel**

-   The “id” I’ve given the Redis Sentinel is “/redis-sentinel” which is equivalent to “redis-sentinel” in the docker-compose.yml file.
-   The Sentinel only gets 3 instances by default. It should be no less than 3, but you can have as many more as you feel is necessary.
-   The Sentinel has both “/redis-master” and “/redis-slave” as dependencies. That means Marathon will not start it until “/redis-master” and “/redis-slave” are running.
-   I’ve set the path to our custom Docker image in the “image” section under “docker”. What value this needs will depend on where you’ve put your built docker images for deployment.
-   The “env” section is where I set the environment variables needed to tell the slave where the master is. The host and port should match the host and port you assigned the master above. More details on this later.

Those are the unique parts of the Marathon configuration that are needed to get the Redis instances to talk to each other. The other parts are just standard settings that anyone who has used Marathon before should immediately recognize.

Slave Configuration
-------------------

Now that we have Marathon configured, we need to prepare our Docker images to use the environment variables that we set above. We will start by configuring the slave instance.

In the test project, I’ve created a folder called “redis-slave” where all of the necessary files reside. I will now go through them one by one. The files in this folder are:

-   Dockerfile
-   redis-entrypoint.sh
-   redis.conf

**Dockerfile**

```dockerfile
FROM redis:6-alpine

RUN mkdir -p /redis

WORKDIR /redis

COPY redis.conf .
COPY redis-entrypoint.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/redis-entrypoint.sh

CMD ["redis-entrypoint.sh"]
```

Here I use one of the official Redis images. The only thing that happens here that deviates from the original Redis image is that I copy the “redis.conf” and “redis-entrypoint.sh” files into the image and then execute the script which looks like this:

**redis-entrypoint.sh**

```shell
#!/bin/sh

MASTER_IP=`getent hosts ${MASTER_HOST} | awk '{ print $1 }'`
MYIP=`getent hosts ${HOST} | awk '{ print $1 }'`

sed -i "s/\$MYIP/$MYIP/g" /redis/redis.conf
sed -i "s/\$PORT0/$PORT0/g" /redis/redis.conf

exec docker-entrypoint.sh redis-server /redis/redis.conf --slaveof ${MASTER_IP} ${MASTER_PORT}
```

The entrypoint script does the following:

1.  It gets the IP address of the master’s hostname that we set as an environment variable in the Marathon configuration.
2.  It gets its own IP address using the “HOST” environment variable that is automatically set by Marathon and contains the Docker image’s own hostname.
3.  It replaces the placeholders in the “redis.conf” file with its own IP address and port (using the “PORT0” environment variable which is also automatically set by Marathon).
4.  It executes the “docker-entrypoint.sh” script which comes with the official Redis image, telling it to start Redis with our configuration file and the “–slaveof” parameter including the IP address and port of the master using the environment variables set in the Marathon configuration. This parameter makes this instance a slave.

**redis.conf**

```shell
slave-announce-ip $MYIP
slave-announce-port $PORT0
```

Our Redis configuration file is short and to the point. “$MYIP” and “$PORT0” are the placeholders that are replaced by the entrypoint script. These two parameters tell Redis to notify all other instances that they have a special IP address and port and is necessary since the instance is in a Docker container. See [the official documentation](http://redis.io/topics/replication#configuring-replication-in-docker-and-nat) for more details.

Sentinel Configuration
----------------------

Now that the slave image is ready to go, we need to setup the Sentinel image. Fortunately, it is similar to the slave configuration. At the root of the project, I’ve created a folder called “redis-sentinel” which contains the following files:

-   Dockerfile
-   sentinel-entrypoint.sh
-   sentinel.conf

**Dockerfile**

```dockerfile
FROM redis:6-alpine

ENV SENTINEL_QUORUM 2
ENV SENTINEL_DOWN_AFTER 1000
ENV SENTINEL_FAILOVER 1000

RUN mkdir -p /redis

WORKDIR /redis

COPY sentinel.conf .
COPY sentinel-entrypoint.sh /usr/local/bin/

RUN chown redis:redis /redis/* &amp;amp;amp;amp;&amp;amp;amp;amp; \
chmod +x /usr/local/bin/sentinel-entrypoint.sh

EXPOSE 26379

ENTRYPOINT ["sentinel-entrypoint.sh"]
```

Again, we use one of the official Redis images as a basis. Next three environment variables are set which will serve as defaults. These can be overwritten in the Marathon configuration if needed. The entrypoint script and configuration file are then copied into the image, the default Sentinel port 26379 is exposed, then we run the entrypoint script.

**sentinel-entrypoint.sh**

```shell
#!/bin/sh

IP=`getent hosts ${HOST} | awk '{ print $1 }'`

sed -i "s/\$SENTINEL_QUORUM/$SENTINEL_QUORUM/g" /redis/sentinel.conf
sed -i "s/\$SENTINEL_DOWN_AFTER/$SENTINEL_DOWN_AFTER/g" /redis/sentinel.conf
sed -i "s/\$SENTINEL_FAILOVER/$SENTINEL_FAILOVER/g" /redis/sentinel.conf
sed -i "s/\$MASTER_HOST/$MASTER_HOST/g" /redis/sentinel.conf
sed -i "s/\$MASTER_PORT/$MASTER_PORT/g" /redis/sentinel.conf
sed -i "s/\$IP/$IP/g" /redis/sentinel.conf
sed -i "s/\$PORT0/$PORT0/g" /redis/sentinel.conf

redis-server /redis/sentinel.conf --sentinel
```

“sentinel-entrypoint.sh” for the most part does the same thing that “redis-entrypoint.sh” did. It gets its own IP address, replaces the placeholders in the configuration file with the real values, then starts a Redis Server instance using the configuration file and the flag “–sentinel” which activates the Sentinel mode.

**sentinel.conf**

```shell
port 26379

dir /tmp

sentinel monitor redismaster $MASTER_HOST $MASTER_PORT $SENTINEL_QUORUM
sentinel down-after-milliseconds redismaster $SENTINEL_DOWN_AFTER
sentinel parallel-syncs redismaster 1
sentinel failover-timeout redismaster $SENTINEL_FAILOVER
sentinel announce-ip $IP
sentinel announce-port $PORT0
```

Not much magic here either. The configuration here is the same as in my article about [Redis Sentinel and Docker Compose](https://blog.alexseifert.com/2016/11/14/using-redis-sentinel-with-docker-compose/), but two new settings have been added which tell the Sentinel instance to announce its IP address and port to the other Redis instances just like the slave. For more details about that, see [the official documentation](http://redis.io/topics/sentinel#sentinel-docker-nat-and-possible-issues).

Conclusion
----------

To sum up what we just did:

-   We configured Marathon to launch four containers, three of which are an instance of Redis.
-   The Redis master is only allowed to have a single instance and has a fixed hostname and port.
-   Environment variables are set for both the Redis slave and Sentinel containers that contain the master’s fixed hostname and port.
-   We created custom Docker images for the slave and Sentinel instances with configurations that tell them to announce their IP address and port to the other instances.

That is more or less all that is needed to get started. Of course several of the configuration details will vary depending on your unique needs, especially for your Marathon configuration, but the basic idea is there.

If you have any questions, comments or suggestions, please feel free to leave them in the comments below.

See the full [example project on GitHub](https://github.com/Developers-Notebook/Redis-Sentinel-Docker-Marathon).

*Note: this article appeared in an older form [here](https://blog.alexseifert.com/2016/11/18/using-redis-sentinel-with-docker-and-marathon/).*