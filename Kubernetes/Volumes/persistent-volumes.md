## Persistent Volumes:

A PersistentVolume is a chunk of storage provided inside the cluster, which can be provided by the administrator or by can be deployed dynamically by employing an ***Storage Class.*** Its a Cluster scoped Resource.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:         # PV will have a specific storage capacity
    storage: 5Gi
  volumeMode: Filesystem  #PersistentVolumes supports **Filesystem** and **Block**.
  accessModes:  # Four types of Modes: ReadWriteOnce, ReadWriteMany, ReadOnlyMany
    - ReadWriteOnce  # and ReadWriteOncePod
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
```

***ReclaimPolicy:***

- Retain -- manual reclamation
- Recycle -- basic scrub (`rm -rf /thevolume/*`)
- Delete -- associated storage asset such as AWS EBS, GCE PD, Azure Disk, or OpenStack Cinder volume is deleted

***Provisioning Volumes:***

There are two ways **PVs** may be provisioned: statically or dynamically.

**Static Provisioning:**

A cluster administrator creates a number of PVs. They carry the details of the real storage, which is available for use by cluster users.

**Dynamic Provisioning:**

When none of the static PVs the administrator created match a user's PersistentVolumeClaim,
the cluster may try to dynamically provision a volume specially for the PVC.
This provisioning is based on StorageClasses: the PVC must request a Storage Class and
the administrator must have created and configured that class for dynamic
provisioning to occur.