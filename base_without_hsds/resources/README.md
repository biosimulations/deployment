# Redis Cluster Config

After starting redis cluster, run the following commands to connect the pods as a cluster. Make sure to change the namespace and root container name.

## Get node IP adresses

for dev:

```
kubectl -n dev get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '
```

for prod

```
kubectl -n prod get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '
```

Make sure to account for a "empty" ip as the last element of the ouput to ignore.

## Run Cluster Configuration on Master Node

Then use the ips as input to the following command. Make sure to set the correct name of the master node based on the environment

For dev:

```
kubectl -n dev exec -it dev-redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 [IP1, IP2...]
```

for prod

```
kubectl -n prod exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 [IP1, IP2...]

```
