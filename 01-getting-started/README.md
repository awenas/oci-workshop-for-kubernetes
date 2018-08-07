# Getting started

Creating a cluster admin role for your user 

```
kubectl create clusterrolebinding cluster-adm \
--clusterrole=cluster-admin \
--user=ocid1.user.oc1..aaaaaaaa3p67n2kmpxnbcnffjow6j5bhe6jze3obob3cjdctfftyfd4zou2q
```

Installing the dashboard 

```
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

Log into the dashboard

```
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```
