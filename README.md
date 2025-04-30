# Elastic
ELK Stack (Elasticsearch, Filebeat, Kibana) to Collect Logs from Applications On Kubernetes Cluster

## Requirements 
- k8s cluster

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
## Deploy Elasticsearch in logging Namespace
```bash
kubectl create namespace elastic
```
```bash
kubectl apply -f elasticsearch.yaml
```

## Verify Elasticsearch Pod & PVC
```bash
kubectl get pods -n elastic```
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
