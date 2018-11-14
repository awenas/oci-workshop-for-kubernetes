# Using Block Storage on OCI Kubernetes 

The OCI Volume Provisioner is responsible for dynamically provisioning block volumes and file systems. If you're using the Oracle managed Kubernetes (Oracle Container Engine) then the provisioner will already be installed on your cluster.

### Create a StorageClass

```yaml
cat <<'$EOF' | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-block-storage
provisioner: oracle.com/oci
$EOF  
```

### Create a PVC

```yaml
cat <<'$EOF' | kubectl create -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-block-volume
spec:
  storageClassName: oci-block-storage
  selector:
    matchLabels:
      failure-domain.beta.kubernetes.io/zone: AD-1
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
$EOF  
```

### Consume storage

The OCI block storage provisioner will create a Flex type volume. In order for these to be attached and consumed you **must** have the OCI Flexvolume Driver installed and configured on your cluster.

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
        claimName: nginx-block-volume
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
