# Things that Work

By running `minikube addons enable storage-provisioner-gluster` you'll get the following:

- daemonset with a glusterfs pod (modified image on quay.io)
- automatic creation of /dev/fake based on environment parameters
- deployment of heketi pod
- ssh-key with private-key in heketi, public key in glusterfs

# Manual execution

```
$ kubectl exec heketi-769698c5f4-9nqln -- heketi-cli --user=admin --secret=minikube cluster create
$ kubectl exec heketi-769698c5f4-9nqln -- heketi-cli --user=admin --secret=minikube node add --cluster=df6b668d934aa3066ea978afd049dd54 --zone=1 --management-host-name=minikube --storage-host-name=172.17.0.1
$ kubectl exec heketi-769698c5f4-9nqln -- heketi-cli --user=admin --secret=minikube device add --node=0dae37a9fe9a02d10394efc3cac2ba2a --name=/dev/fake
```

**yuck** need to delete/recreate the storage-class with heketi IP instead of hostname

```
$ kubectl get ep/heketi
$ kubectl get sc/glusterfile -oyaml > /var/tmp/sc.yaml
$ vim /var/tmp/sc.yaml
$ kubectl delete sc/glusterfile
$ kubectl apply -f /var/tmp/sc.yaml
```

```
$ kubectl apply -f /var/tmp/claim.yaml
$ kubectl describe pvc/claim1
```

# TODO

- use coredns addon with hostname heketi.default.svc.cluster.local in the StorageClass (default=namespace)
- send PRs for standard glusterfs-server containers with fake-device support, public ssh-key merging
