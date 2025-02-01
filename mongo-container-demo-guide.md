# Steps for docker demonstration.

Download mongo image from docker hub.
$ docker pull mongo

Download mongo-express image from docker hub.

$ docker pull mongo-express

 Create a network for mongo containers.
$ docker network create mongo-network

# Verify network is now present.

$ docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
e2e3dadbb67a   bridge          bridge    local
9ebb1cd7a64d   host            host      local
add958f9fa02   mongo-network   bridge    local
2702035c42a3   none            null      local


Reference: https://hub.docker.com/_/mongo/

# Mongo database container deployment "basic" workflow.
1. Create container for mongo database in detached mode.
2. Set local port on your host system running docker to 27017 and bind to mongo db's port which is also 27017.
3. Add container name for example demo-mongodb.
4. Identify the network to be used by mongodb for connections as mongo-network.
5. Add environment variables for the "root" username and password.
6. Identify docker image to use which in this case is named mongo.
7. For data storage you can create a local folder on the host system that is running docker e.g. /home/<username>/datadir. 


# This is just a demonstration so the example I provided so a typical data directory will be utilized versus "docker volumes".
To add a basic folder just use the "-v" option for volume and a path to a folder that you have permission to write to on your localhost filesystem e.g. "-v /datadir:/data/db"

docker run -d -p 27017:27017 --name demo-mongodb \
--network mongo-network \
-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
-e MONGO_INITDB_ROOT_PASSWORD=secret \
mongo

# Verify that the container is up and running.
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                      NAMES
12068f916903   mongo     "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:27017->27017/tcp   demo-mongodb

# We go further and confirm the system is running:
$ docker logs demo-mongodb | head

about to fork child process, waiting until server is ready for connections.
forked process: 27

{"t":{"$date":"2025-01-26T12:13:25.237+00:00"},"s":"I",  "c":"CONTROL",  "id":20698,   "ctx":"main","msg":"***** SERVER RESTARTED *****"}
{"t":{"$date":"2025-01-26T12:13:25.241+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
{"t":{"$date":"2025-01-26T12:13:25.242+00:00"},"s":"I",  "c":"CONTROL",  "id":5945603, "ctx":"main","msg":"Multi threading initialized"}
{"t":{"$date":"2025-01-26T12:13:25.245+00:00"},"s":"I",  "c":"NETWORK",  "id":4648601, "ctx":"main","msg":"Implicit TCP FastOpen unavailable. If TCP FastOpen is required, set at least one of the related parameters","attr":{"relatedParameters":["tcpFastOpenServer","tcpFastOpenClient","tcpFastOpenQueueSize"]}}
{"t":{"$date":"2025-01-26T12:13:25.246+00:00"},"s":"I",  "c":"NETWORK",  "id":4915701, "ctx":"main","msg":"Initialized wire specification","attr":{"spec":{"incomingExternalClient":{"minWireVersion":0,"maxWireVersion":25},"incomingInternalClient":{"minWireVersion":0,"maxWireVersion":25},"outgoing":{"minWireVersion":6,"maxWireVersion":25},"isInternalClient":true}}}
{"t":{"$date":"2025-01-26T12:13:25.247+00:00"},"s":"I",  "c":"TENANT_M", "id":7091600, "ctx":"main","msg":"Starting TenantMigrationAccessBlockerRegistry"}
{"t":{"$date":"2025-01-26T12:13:25.247+00:00"},"s":"I",  "c":"CONTROL",  "id":4615611, "ctx":"initandlisten","msg":"MongoDB starting","attr":{"pid":27,"port":27017,"dbPath":"/data/db","architecture":"64-bit","host":"12068f916903"}}

# Create a mongo-express container to connect to the mongo db container.
Reference: https://hub.docker.com/_/mongo-express/

docker run -d \
-p 8081:8081 \
--name mongo-express \
--network mongo-network \
-e ME_CONFIG_MONGODB_SERVER="demo-mongodb" \
-e ME_CONFIG_MONGODB_ADMINUSERNAME="mongoadmin" \
-e ME_CONFIG_MONGODB_ADMINPASSWORD="secret" \
mongo-express

# Verify we have both containers running:
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                      NAMES
d7bbdc485c78   mongo-express   "/sbin/tini -- /dock…"   4 seconds ago    Up 4 seconds    0.0.0.0:8081->8081/tcp     mongo-express
12068f916903   mongo           "docker-entrypoint.s…"   28 minutes ago   Up 28 minutes   0.0.0.0:27017->27017/tcp   demo-mongodb

# Verify status of mongo-express container.
$ docker logs mongo-express
Waiting for mongo:27017...

Welcome to mongo-express 1.0.2
------------------------


Mongo Express server listening at http://0.0.0.0:8081
Server is open to allow connections from anyone (0.0.0.0)
basicAuth credentials are "admin:pass", it is recommended you change this in your config.js!
