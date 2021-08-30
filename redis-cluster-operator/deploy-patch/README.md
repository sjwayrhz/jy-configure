Create a new namespace called : redis-cluster
```
$ kubectl create ns redis-cluster
```
Register the DistributedRedisCluster and RedisClusterBackup custom resource definition (CRD).
```
$ kubectl create -f deploy-patch/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy-patch/crds/redis.kun_redisclusterbackups_crd.yaml
```
use namespace-scoped
```
$ kubectl create -f deploy-patch/service_account.yaml
$ kubectl create -f deploy-patch/namespace
```

Deploy a sample Redis Cluster
```
$ kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml
```