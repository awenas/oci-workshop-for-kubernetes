

Deploy the block storage provisioner

```yaml
cat <<'$EOF' | kubectl create -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: oci-block-storage-provisioner
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

Create a storage class

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


