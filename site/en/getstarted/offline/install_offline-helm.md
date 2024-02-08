---
id: install_offline-helm.md
summary: Learn how to install Milvus on Kubernetes offline.
---

# Install Milvus Offline with Helm Charts

This topic describes how to install Milvus with Helm charts in an offline environment. 

Installation of Milvus might fail due to image loading errors. You can install Milvus in an offline environment to avoid such problem.

## Download files and images

To install Milvus offline, you need to pull and save all images in an online environment first, and then transfer them to the target host and load them manually.

1. Add and update Milvus Helm repository locally.

```
helm repo add milvus https://zilliztech.github.io/milvus-helm/
helm repo update
```

2. Get a Kubernetes manifest.

- For Milvus standalone:

```
helm template my-release --set cluster.enabled=false --set etcd.replicaCount=1 --set minio.mode=standalone --set pulsar.enabled=false milvus/milvus > milvus_manifest.yaml
```

- For Milvus cluster:

```cluster
helm template my-release milvus/milvus > milvus_manifest.yaml
```
Note that the cluster release needs a [K8s service](https://milvus.io/docs/install_cluster-milvusoperator.md#Create-a-K8s-Cluster), [Minikube](https://minikube.sigs.k8s.io/docs/) is recommended  for a quick start. 

If you want to change multiple configurations, you can download a [`value.yaml`](https://github.com/milvus-io/milvus-helm/blob/master/charts/milvus/values.yaml) file, specify configurations in it, and generate a manifest based on it.

```bash
wget https://raw.githubusercontent.com/milvus-io/milvus-helm/master/charts/milvus/values.yaml
helm template -f values.yaml my-release milvus/milvus > milvus_manifest.yaml
```

3. Download requirement and script files.

```
$ wget https://raw.githubusercontent.com/milvus-io/milvus/master/deployments/offline/requirements.txt
$ wget https://raw.githubusercontent.com/milvus-io/milvus/master/deployments/offline/save_image.py
```

4. Pull and save images.

```
pip3 install -r requirements.txt
python3 save_image.py --manifest milvus_manifest.yaml
```

<div class="alert note">
The images are stored in the <code>/images</code> folder.
</div>

5. Load the images.

```
cd images/for image in $(find . -type f -name "*.tar.gz") ; do gunzip -c $image | docker load; done
```

## Install Milvus offline

Having transferred the images to the target host, run the following command to install Milvus offline.

```
kubectl apply -f milvus_manifest.yaml
```

Run the following command to check the current status of Milvus pods.

```bash
kubectl get pods
```

```
NAME                                            READY   STATUS            RESTARTS   AGE
my-release-etcd-0                               0/1     Running           0          2m14s
my-release-etcd-1                               0/1     Running           0          2m14s
my-release-etcd-2                               0/1     Running           0          2m14s
my-release-milvus-datacoord-6cc5c78447-2bl9h    0/1     Running           0          2m14s
my-release-milvus-datanode-6c6946fb56-xnx4v     0/1     Running           0          2m14s
my-release-milvus-indexcoord-777b8f4766-4tghb   0/1     Running           0          2m14s
my-release-milvus-indexnode-7dbd8f476f-kpgtl    0/1     PodInitializing   0          2m14s
my-release-milvus-proxy-75c4dd96b-5ndwf         0/1     PodInitializing   0          2m14s
my-release-milvus-querycoord-5b67695b5f-6tv7d   0/1     PodInitializing   0          2m14s
my-release-milvus-querynode-b49bdb475-tht6k     0/1     PodInitializing   0          2m14s
my-release-milvus-rootcoord-6f5998b4f5-4wdz6    0/1     PodInitializing   0          2m14s
my-release-minio-0                              1/1     Running           0          2m14s
my-release-minio-1                              1/1     Running           0          2m14s
my-release-minio-2                              1/1     Running           0          2m14s
my-release-minio-3                              1/1     Running           0          2m14s
my-release-pulsar-bookie-0                      0/1     Init:0/1          0          2m14s
my-release-pulsar-bookie-1                      0/1     Init:0/1          0          2m13s
my-release-pulsar-bookie-2                      0/1     Init:0/1          0          2m13s
my-release-pulsar-bookie-init-4jb74             0/1     Init:0/1          0          2m13s
my-release-pulsar-broker-0                      0/1     Init:0/2          0          2m14s
my-release-pulsar-proxy-0                       0/1     Init:0/2          0          2m14s
my-release-pulsar-pulsar-init-vmr8d             0/1     Init:0/2          0          2m13s
my-release-pulsar-recovery-0                    0/1     Init:0/1          0          2m14s
my-release-pulsar-zookeeper-0                   1/1     Running           0          2m13s
my-release-pulsar-zookeeper-1                   0/1     Running           0          8s
```

## Uninstall Milvus

To uninstall Milvus, run the following command.

```
kubectl delete -f milvus_manifest.yaml
```

## What's next

Having installed Milvus, you can:

- Check [Hello Milvus](example_code.md) to run an example code with different SDKs to see what Milvus can do.

- Learn the basic operations of Milvus:
  - [Connect to Milvus server](manage_connection.md)
  - [Create a collection](create_collection.md)
  - [Create a partition](create_partition.md)
  - [Insert data](insert_data.md)
  - [Conduct a vector search](search.md)

- [Upgrade Milvus Using Helm Chart](upgrade_milvus_cluster-helm.md).
- [Scale your Milvus cluster](scaleout.md).
- Explore [MilvusDM](migrate_overview.md), an open-source tool designed for importing and exporting data in Milvus.
- [Monitor Milvus with Prometheus](monitor.md).
