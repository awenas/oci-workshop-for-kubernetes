# Using Block Storage on OCI Kubernetes 

The OCI Volume Provisioner is responsible for dynamically provisioning block volumes and file systems. If you're using the Oracle managed Kubernetes (Oracle Container Engine) then the provisioner will already be installed on your cluster.

### Deploy

The following is only needed if you're running a self managed Kubernetes cluster on OCI.

```yaml
cat <<'$EOF' | kubectl create -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: oci-block-volume-provisioner
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: oci-volume-provisioner
    spec:
      serviceAccountName: oci-volume-provisioner
      containers:
        - name: oci-volume-provisioner
          image: iad.ocir.io/oracle/oci-volume-provisioner:0.9.0
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: PROVISIONER_TYPE
              value: oracle.com/oci
          volumeMounts:
            - name: config
              mountPath: /etc/oci/
              readOnly: true
      volumes:
        - name: config
          secret:
            secretName: oci-volume-provisioner
$EOF
```

### Create a StorageClass

```yaml
cat <<'$EOF' | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-block-storage
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
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
  selector:
    matchLabels:
      failure-domain.beta.kubernetes.io/zone: PHX-AD-1
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
$EOF  
```

### Consume storage

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
