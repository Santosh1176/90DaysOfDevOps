## Persistent Volume Claim

To declare a binding between the Persistent Volume created earlier, we need o create a Claim for thet PV. This is achieved using a Resource known as **Persistent Volume Claim.** If the PersistentVolume exists and has not reserved ***PersistentVolumeClaims*** through its `claimRef` field, then the *PersistentVolume* and *PersistentVolumeClaim* will be bound. 

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-pv
spec:
  storageClassName: ""
  claimRef:
    name: foo-pvc
    namespace: foo
```

To guarantee any binding privileges to the *PersistentVolume*. If other 
*PersistentVolumeClaims* could use the PV that you specify, we first need
 to reserve that storage volume by specifying the relevant *PersistentVolumeClaim* in the `claimRef`  field of the PV so that other PVCs can not bind to it.

## Persistent Volume Plugins:

PersistentVolume types can also be implemented as plugins. Check the [Kubernetes Link](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes) to see the full list of plugins supperted by Kubernetes.