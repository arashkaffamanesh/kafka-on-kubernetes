# Restore Kafka from OpenEBS Snapshots, the Hard or the Easy Way?

This is an attempt to create a clone from the kafka1 cluster into a new kafka clutser in the namespace kafka1-clone on the same k8s cluster (Rancher Kubernetes) with OpenEBS Snapshots.

The steps are as following:

Create a snapshot from all PVCs in Kafka1 namespace.

Create a new namespace kafka-clone.

Create PVCs from the snapshots (with the same name schema: data-kafka1-kafka-0, data-kafka1-zookeeper-0, etc..)

Run a new cluster with strimzi in the kafka1-clone namespace.

Expected result:

The kafka brokers and zookeeper pods should bind to the cloned PVCs and have the same data as in kafka1 cluster.

## Create kafka1-clone namespace

```bash
k create ns kafka1-clone
```

## Create the snapshots from brokers and zookeepers:

```bash
k create -f snapshot-data-kafka1-kafka-0.yaml
k create -f snapshot-data-kafka1-kafka-1.yaml
k create -f snapshot-data-kafka1-kafka-2.yaml
k create -f snapshot-data-kafka1-zookeeper-0.yaml
k create -f snapshot-data-kafka1-zookeeper-1.yaml
k create -f snapshot-data-kafka1-zookeeper-2.yaml
```

## List the volumesnapshots

```bash
k get volumesnapshot
```

## List the volumesnapshotdata

```bash
k get volumesnapshotdata
```

## Create new PVCs in the kafka1-clone namespace

```bash
k apply -f pvc-snapshot-data-kafka1-kafka-0.yaml -n kafka1-clone
k apply -f pvc-snapshot-data-kafka1-kafka-1.yaml -n kafka1-clone
k apply -f pvc-snapshot-data-kafka1-kafka-2.yaml -n kafka1-clone
k apply -f pvc-snapshot-data-kafka1-zookeeper-0.yaml -n kafka1-clone
k apply -f pvc-snapshot-data-kafka1-zookeeper-1.yaml -n kafka1-clone
k apply -f pvc-snapshot-data-kafka1-zookeeper-2.yaml -n kafka1-clone
```

## Get the PVCs

```bash
$ k get pvc -n kafka1-clone
```

## Allow to deploy kafka1 cluster in namespace kafka1-clone

```bash
helm upgrade --reuse-values --set watchNamespaces="{kafka1-clone}" strimzi-cluster-operator strimzi/strimzi-kafka-operator
```

## Deploy a clone of kafka1 cluster in the kafka1-clone namespace

```bash
$ k apply -f kafka1-clone-cluster.yaml
```