#  DEMO for GKE configuration and scaling

## Setup Account Info
```
gcloud info | grep Account
gcloud config set project cockroach-glenn
```

## Create Cluster /w 6 nodes
This can be GKE, EKS, Openshift, Vsphere, ..
```
gcloud container clusters create cockroachdb --machine-type n1-standard-16  --num-nodes 6

## Restart if shrunk down
#gcloud container clusters resize cockroachdb --num-nodes=0
gcloud container clusters resize cockroachdb --num-nodes=6

## List clusters
gcloud container clusters list
```

## Create /w 3 nodes -- kubectl
```
#curl -O https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/bring-your-own-certs/cockroachdb-statefulset.yaml
kubectl create -f cockroachdb-statefulset.yaml 

kubectl get persistentvolumes

kubectl create -f https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/cluster-init.yaml

kubectl get job cluster-init

kubectl get pods
```
**Connect SQL shell and restore DATABASE ::**
```
kubectl run cockroachdb -it --image=cockroachdb/cockroach:v20.1.4 --rm --restart=Never -- sql --insecure --host=cockroachdb-public

-- Install / Enable License

SQL>
    SET CLUSTER SETTING cluster.organization = "Cockroach Labs - Production Testing";
    SET CLUSTER SETTING enterprise.license = "crl-0-EJL04ukFGAEiI0NvY2tyb2FjaCBMYWJzIC0gUHJvZHVjdGlvbiBUZXN0aW5n";

-- Restore backup of Database
    restore database tpcc from 'gs://querylabs/backup_lab1';
```

## Forward one of the ports for AdminUI -- kubectl

```
kubectl port-forward cockroachdb-2 8080
```
Monitor with AdminUI:
+ Show Restore Job
+ Show HW usage
+ Show Replication
    + LeaseHolders per Node
    + Replicas per Node

## Scale up Stateful Set
Double the number of nodes for the Cluster *statefulset*.
```
kubectl scale statefulset cockroachdb --replicas=6

kubectl get pods
```
Monitor with AdminUI:
+ Show New Nodes 
+ Show Replication Re-Balance
    + LeaseHolders per Node
    + Replicas per Node

**Kill a node/pod ::**
```
kubectl delete pod cockroachdb-4
```
AdminUI:
+ Show Failed Node
+ Show Restart 

**GetPods to show Restart ::**
```
kubectl get pods
```

## SSH to Containers / instances (misc)
```
gcloud compute config-ssh
gcloud compute instances list
ssh gke-cockroachdb-default-pool-1527ec03-1j35.us-east1-b.cockroach-glenn
```