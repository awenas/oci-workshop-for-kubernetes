# Using File Storage on OCI Kubernetes

First we need to deploy the OCI Volume Provisioner in FSS mode. We do this by setting the PROVISIONER_TYPE to `oracle.com/oci-fss`

```yaml
cat <<'$EOF' | kubectl create -f -
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: oci-volume-provisioner
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

Create a new Storage Class

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-fss
provisioner: oracle.com/oci-fss
```

You can optionally choose to provide a subnetId in the Storage Class parameters such that mount targets 
will be provisioned on this subnet.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-fss
provisioner: oracle.com/oci-fss
parameters:
  subnetId: {{SUBNET_OCID}}
```

Additionally you can specify a specific mount target in the storage class also if you already have a mount target to use for FSS volumes.


```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: oci-fss
provisioner: oracle.com/oci-fss
parameters:
  mntTargetId: {{MNT_TARGET_OCID}}
```
