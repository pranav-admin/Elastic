# Elastic
ELK Stack (Elasticsearch, Filebeat, Kibana) to Collect Logs from Applications On Kubernetes Cluster

## Requirements 
- k8s cluster


## Using StorageClass to Dynamically provision a volume (Optional)
### Install NFS Server utilities on Control Plane and Client utilities on Worker nodes
```bash
 dnf install nfs-utils* -y
```
```bash
 mkdir /app
```
```bash
chmod 777 /app
```
```bash
vim /etc/exports

     /app *(rw,sync)
```

```bash
systemctl enable --now nfs-server
```
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=master --set nfs.path=/app
```
```bash
kubectl get storageclass
```

## Deploy Sample Applications in web-app Namespace
```bash
kubectl create namespace web-apps
```

```bash
 kubectl apply -f web1.yaml
```

```bash
kubectl apply -f web2.yaml
```
## Deploy Elasticsearch in elastic Namespace
```bash
kubectl create namespace elastic
```
```bash
kubectl apply -f elasticsearch.yaml
```

## Verify Elasticsearch Pod & PVC
```bash
kubectl get pods -n elastic
```
```bash
kubectl get pvc -n elastic
```

```bash 
kubectl get pv -n elastic
```


## Deploy Kibana in elastic Namespace
```bash 
kubectl apply -f kibana.yaml
```

## Verify Kibana

```bash 
kubectl get pods -n elastic
```

## Deploy Filebeat as a DaemonSet
### Download Filebeat Kubernetes Manifest from Elastic

```bash 
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.17/deploy/kubernetes/filebeat-kubernetes.yaml
```

## Modify filebeat-kubernetes.yaml file

   Update ELASTICSEARCH_HOST with = http://elasticsearch.elastic.svc.cluster.local:9200
   Add the namespace "elastic" to the Filebeat pod annotations if you want namespace-specific logs.
   But Filebeat is running as a DaemonSet to it has access to all pods across all namespaces via its ClusterRole.```

## Deploy Filebeat

```bash 
kubectl apply -f filebeat-kubernetes-updated.yaml
```

## On Kibana

   Explore on My Own
    Click Home Left Panel
    Go to Stack Management
    Click Index Patterns - create an index pattern name e.g: filebeat-*
    Select @timestamp in Timestamp field
    Click create index pattern.
    Go to Discover on the left panel of homepage to see logs from app1 and app2.

## Verify the Logs from the apps

```bash
kubectl logs web1 -n web-apps | tail
```

```bash 
kubectl logs web2 -n web-apps | head
```

## Verify it from Kibana UI

   On the left panel (under Available fields)
    Scroll down to the bottom to see e.g. log message, log.file.path, etc.
    Click to examine them.

## Now, Deploy and Log an NGINX application

```bash
kubectl apply -f nginx-deployment.yaml
```


## Create some Filters in Kibana to examine Nginx logs

    Add filter
    Field = kubernetes.labels.app, Operator = is, Value = nginx & Save. (You may have to change timestamp next to the "Refresh button" to see some logs)
