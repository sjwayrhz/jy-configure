Register the DistributedRedisCluster and RedisClusterBackup custom resource definition (CRD).
```
$ kubectl create -f deploy-patch/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy-patch/crds/redis.kun_redisclusterbackups_crd.yaml
```
use namespace-scoped
```
$ kubectl create -f deploy-patch/service_account.yaml
$ kubectl create -f deploy-patch/namespace/role.yaml
$ kubectl create -f deploy-patch/namespace/role_binding.yaml
$ kubectl create -f deploy-patch/namespace/operator.yaml
```

Deploy a sample Redis Cluster
```
$ kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml
```