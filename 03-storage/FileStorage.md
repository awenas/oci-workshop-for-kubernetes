# Using File Storage on OCI Kubernetes

There are two ways to use File Storage service within Kubernetes on OCI. You can either 

1. Manually create a file system and treat it as an NFS volume type (see [here](examples/fss-manual.yaml) for an example).
2. Use a plugin to dynamically create file systems for you automatically that exist behind a mount target.

### Install

First we need to install the OCI Volume Provisioner in FSS mode if it's not already installed in your cluster. 

https://github.com/oracle/oci-cloud-controller-manager/tree/master/manifests/volume-provisioner

### Create a mount target

```
oci fs mount-target create --availability-domain=UpwH:PHX-AD-1 \
--compartment-id=ocid1.compartment.oc1..aaaaaaaa... \
--subnet-id=ocid1.subnet.oc1.eu-frankfurt-1.aaaaaaaa...
```

### Create a storage class

```yaml
cat <<'$EOF' | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-fss
provisioner: oracle.com/oci-fss
parameters:
  mntTargetId: ocid1.mounttarget.oc1.phx.aaaaaa4np2soafcrobuhqllqojxwiotqnb4c2ylefuyqaaaa
$EOF  
```

### Create a PVC

Next we create a PVC. Note that, although we need to provide a storage capacity in the request, this is not used for FSS file systems and exists purely to make Kubernetes happy since resources.requests.storage is a required field in a PVC.

```yaml
cat <<'$EOF' | kubectl create -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-fss-volume
spec:
  storageClassName: oci-fss
  # The following selector controls which AD the file system is provisioned in.
  selector:
    matchLabels:
      failure-domain.beta.kubernetes.io/zone: AD-1
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
$EOF
```

Check if the PVC is bound

```sh
❯ kubectl get pvc
```

### Consume the PVC storage in a Pod

Now that we have an FSS file system bound to our PVC, we can reference it in a Pod.

```yaml
cat <<'$EOF' | kubectl create -f -
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  volumes:
    - name: nginx
      persistentVolumeClaim:
        claimName: nginx-fss-volume
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
          name: http
      volumeMounts:
      - mountPath: /usr/share/nginx/html
        name: nginx
$EOF
```

Ensure that the Pod is running correctly 

```sh
❯ kubectl get po

NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          7m
```

Data written to /usr/share/nginx/html will be persisted to the underlying OCI file system.

