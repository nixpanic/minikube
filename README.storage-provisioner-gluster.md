# Things that Work

By running `minikube addons enable storage-provisioner-gluster` you'll get the following:

- daemonset with a glusterfs pod (modified image on quay.io)
- automatic creation of /dev/fake based on environment parameters
- deployment of heketi pod
- ssh-key with private-key in heketi, public key in glusterfs
- ConfigMap with the initial topology.json
- automatically load the topology.json in case there is no heketi.db yet
- create a StorageClass called glusterfile

# Manual execution

**yuck** need to delete/recreate the storage-class with heketi IP instead of hostname

```
$ kubectl get ep/heketi
$ kubectl get sc/glusterfile -oyaml > /var/tmp/sc.yaml
$ vim /var/tmp/sc.yaml
$ kubectl delete sc/glusterfile
$ kubectl apply -f /var/tmp/sc.yaml
```

```
$ cat << EOF | kubectl apply -f -
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim1
  annotations:
    volume.beta.kubernetes.io/storage-class: "glusterfile"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
EOF

$ kubectl describe pvc/claim1
```

# TODO

- drop the ssh-key bit, and use kubexec method with [proper rbac stuff](https://github.com/heketi/heketi/blob/master/docs/admin/install-kubernetes.md)
  The heketi pod restarts once, is that the cause of the missing rbac rules?
  `Error creating: pods "heketi-947f67646-" is forbidden: error looking up service account default/heketi-service-account: serviceaccount "heketi-service-account" not found`
- drop public ssh-key merging
- hostname heketi.default.svc.cluster.local in the StorageClass does not get resolved from within the storage-provisioner (needs rbac?)
- send PRs for standard glusterfs-server containers with fake-device support
- send PR for heketi-start.sh that imports the topology.json upon first start
