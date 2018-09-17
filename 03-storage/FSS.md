# Using File Storage on OCI Kubernetes

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
