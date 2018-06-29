# Storage

Like most public clouds, Oracle Cloud Infrastructure supports three distinct types of storage. File, Block and Object. 

This section will discuss how we can use these types of storage inside a Kubernetes cluster.

## Block Storage

See https://docs.cloud.oracle.com/iaas/Content/Block/Concepts/overview.htm

Block volumes can be attached to Kubernetes nodes using the oci-volume-provisioner. Block volumes are attached to a node using iSCSI. The obvious thing to note about block storage inside Kubernetes is that a the block volumes must be attached to the same Kubernetes Node as the Pod is running on. This means that if you have 5 Pods, they must all run on a single node. 

Block Volumes are only accessible to instances in the same availability domain. You cannot move a volume between availability domains or regions.

## File Storage (FSS)

See https://docs.cloud.oracle.com/iaas/Content/File/Concepts/filestorageoverview.htm
