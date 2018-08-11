# Things that Work

By running `minikube addons enable storage-provisioner-gluster` you'll get the following:

- daemonset with a glusterfs pod (modified image on quay.io)
- automatic creation of /dev/fake based on environment parameters
- deployment of heketi pod and service-account
- ConfigMap with the initial topology.json
- automatically load the topology.json in case there is no heketi.db yet
- create a StorageClass called glusterfile with external storage provider

# Manual execution

Follow the [minikube installation guidelines](...), mainly for the docker and
hypervisor drivers. Make sure that `minikube start ...` works.

Still need to build `minikube` from source:

```
$ go get k8s.io/minikube
$ cd $GOPATH/k8s.io/minikube
$ git remote add nixpanic https://github.com/nixpanic/minikube
$ git checkout -t -b addons/gluster nixpanic/addons/gluster
$ make
```

Start the `minikube` Kubernetes cluster:

```
$ ./out/minikube start --network-plugin=cni --vm-driver=kvm2 && ./out/minikube addons enable storage-provisioner-gluster
```

Once the cluster is up and running, create a PVC:

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

$ kubectl get pvc/claim1 -owide
```

It may take a while until the 1st PVC gets into `Bound` status. There are
several containers that need to get started and configured.


# TODO

- send PRs for standard glusterfs-server containers with fake-device support
- [get the PR merged](https://github.com/heketi/heketi/pull/1314) for heketi-start.sh that imports the topology.json upon first start
