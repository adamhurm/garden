---
created: 2024-10-22T14:08
updated: 2025-07-21T17:36
tags:
  - kubernetes
  - longhorn
  - nfs
  - pvc
---

1. Create **myapp-target** pvc manifest. Set StorageClass to **longhorn**.

2. Make sure our **myapp-source** pvc is not in use:
   `k scale deploy -n example myapp --replicas=0`

3. Use the [volume migration job](https://github.com/longhorn/longhorn/blob/master/examples/data_migration.yaml): `k apply -f data_migration.yaml`
   namespace=**example**, source=**myapp-pvc**, target=**myapp-target**

4. Check if job is complete:
   `k describe job volume-migration -n example`

5. Delete job once it is no longer needed:
   `k delete job volume-migration -n example`

6. Test new pvc:
   `k scale deploy -n example myapp --replicas=1`

7. If everything is good, clean up old pvc:
   `k delete pvc -n example myapp-source`

