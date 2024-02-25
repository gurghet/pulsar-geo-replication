# pulsar-geo-replication
Simple minimal demonstration of topic geo replication

## Commands

```docker-compose up```

You will have two clusters. They will be in the same network to use the Docker provided DNS. Also for simplicity no keys are generated and communication is plaintext.

The first cluster `cluster-a` will be available from the host on port `8080`, `cluster-b` will be available from port `8081`.

```
./pulsar-admin --admin-url http://localhost:8080 cluster create cluster-b --broker-url pulsar://broker-edge1:6650 --url http://broker-edge1:8080
```
This connects `cluster-a` to `cluster-b`, the following does the other way around:
```
./pulsar-admin --admin-url http://localhost:8081 clusters create cluster-a --broker-url pulsar://broker:6650 --url http://broker:8080
```
Now we need to add the `tenant/namespace` in both clusters.
```
./pulsar-admin --admin-url http://localhost:8080 tenants create edge1 --allowed-clusters cluster-a,cluster-b
./pulsar-admin --admin-url http://localhost:8081 tenants create edge1 --allowed-clusters cluster-a,cluster-b
./pulsar-admin --admin-url http://localhost:8080 namespaces create edge1/replicated --clusters cluster-a,cluster-b
./pulsar-admin --admin-url http://localhost:8081 namespaces create edge1/replicated --clusters cluster-a,cluster-b
```

Now, by default, any topic created in the `edge1/replicated` namespace will be replicated in both clusters. Let's create an `event` topic there.
```
./pulsar-admin --admin-url http://localhost:8080 topics create edge1/replicated/events
```

Open a consumer for both cluster to check if the messages will come through:
```
./pulsar-client --url http://localhost:8080 consume --subscription-name "sub-a" persistent://edge1/replicated/events
```
and in another terminal
```
./pulsar-client --url http://localhost:8081 consume --subscription-name "sub-b" persistent://edge1/replicated/events
```
and in a third
```
./pulsar-client --url http://localhost:8080 produce persistent://edge1/replicated/events --messages "Hello world\!"
```
You will see the same message in both clusters.