# Using File Storage on OCI Kubernetes

### Install

First we need to install the OCI Volume Provisioner in FSS mode if it's not already installed in your cluster. Newer versions of OKE (Oracle Container Engine) may install the oci-fss-volume-provisioner as part of your cluster. 

Ensure that you set PROVISIONER_TYPE to `oracle.com/oci-fss`

```yaml
cat <<'$EOF' | kubectl create -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: oci-fss-volume-provisioner
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
              value: oracle.com/oci-fss
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

### Create a new Storage Class

There are two ways to configure the StorageClass for FSS. Either you can specify a `subnetId` parameter and have the provisioner dynamically create a new mount target, or, you can specify an existing mount target OCID.

#### Dynamically provision a new MountTarget

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-fss
provisioner: oracle.com/oci-fss
parameters:
  subnetId: {{SUBNET_OCID}}
```

#### Use an existing Mount Target

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-fss
provisioner: oracle.com/oci-fss
parameters:
  mntTargetId: {{MNT_TARGET_OCID}}
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
      failure-domain.beta.kubernetes.io/zone: PHX-AD-2
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
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

