# Installing Prometheus on OKE

The following guide will walk you through how to install Prometheus on OKE.

Firstly, make sure that you have tiller installed in your Kubernetes cluster.

```
helm install --name prometheus-prod stable/prometheus
```

This will create two block volumes on OCI by default. 

To access the dashboard run:

```
export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9090
```

Accessing Alert Manager

```
export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=alertmanager" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093
```
