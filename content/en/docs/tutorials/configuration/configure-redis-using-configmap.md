---
reviewers:
- eparis
- pmorie
title: Configuring Redis using a ConfigMap
content_type: tutorial
weight: 30
---

<!-- overview -->

This tutorial provides a working example of how to configure Redis using a [ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/). 

Redis is an open-source server that stores data in memory. It can be used as an application cache or quick-response database. For more information, visit the [Redis](https://redis.io/about/) webpage.

## {{% heading "objectives" %}}


* Create a ConfigMap with Redis configuration values
* Create a Redis Pod that mounts the created ConfigMap
* Add configuration values to the ConfigMap
* Verify that the configuration was correctly applied



## {{% heading "prerequisites" %}}

You will need:
  * Understanding of how to [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
  * A Kubernetes cluster
  * `kubectl` CLI (ver 1.14+)
    * Must be configured to the cluster
    * Should have 2 nodes not acting as control pane hosts

**Note:** If you do not already have a cluster, you can create one using [minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) or the [Killercoda](https://killercoda.com/playgrounds/scenario/kubernetes) Kubernetes playground.




<!-- lessoncontent -->


## Create a ConfigMap with Redis configuration values

Create a ConfigMap with an empty configuration block:

```shell
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```

## Create a Redis pod that mounts the ConfigMap

1. Apply the ConfigMap created above:

```shell
kubectl apply -f example-redis-config.yaml
```

2. Apply the Redis pod manifest:

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

This will generate the following YAML file:

{{% code_sample file="pods/config/redis-pod.yaml" %}}

* A volume named `config` is created by `spec.volumes[1]`
* The `key` and `path` under `spec.volumes[1].configMap.items[0]` exposes the `redis-config` key from the 
  `example-redis-config` ConfigMap as a file named `redis.conf` on the `config` volume.
* The `config` volume is then mounted at `/redis-master` by `spec.containers[0].volumeMounts[1]`.

These configuration changes expose `data.redis-config` from the `example-redis-config` ConfigMap. In the Redis pod, the data is represented as `/redis-master/redis.conf`.

## Add configuration values to the ConfigMap

1. Add `maxmemory 2mb` and `maxmemory-policy allkeys-lru` to the `redis-config` element of the `example-redis-config` ConfigMap:

{{% code_sample file="pods/config/example-redis-config.yaml" %}}

2. Apply the updated ConfigMap:

```shell
kubectl apply -f example-redis-config.yaml
```

3. Confirm that the ConfigMap was updated:

```shell
kubectl describe configmap/example-redis-config
```

You should see the configuration values we just added:

```shell
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru
```

## Verify that the Redis configuration was correctly applied

Though the ConfigMap has been updated, the Redis pod must be restarted before the configuration values update.

1. Delete the pod:

```shell
kubectl delete pod redis
```

2. Recreate the pod:

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

3. Check the configuration values:

```shell
kubectl exec -it redis -- redis-cli
```

4. Check `maxmemory`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory
```

It should now return the updated value of `2097152` (2MB):

```shell
1) "maxmemory"
2) "2097152"
```

5. Check `maxmemory-policy`:

```shell
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

It now reflects the desired value of `allkeys-lru`:

```shell
1) "maxmemory-policy"
2) "allkeys-lru"
```

## {{% heading "whatsnext" %}}


* Learn more about [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/).
* Follow an example of [Updating configuration via a ConfigMap](/docs/tutorials/configuration/updating-configuration-via-a-configmap/).
