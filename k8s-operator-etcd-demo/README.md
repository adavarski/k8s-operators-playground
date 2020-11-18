### k8s operator etcd : demo

#### Deploy
```
$ kubectl describe clusterrole cluster-admin
$ kubectl create -f etcd-operator-crd.yaml
$ kubectl get crd
$ kubectl create -f etcd-operator-sa.yaml
$ kubectl get serviceaccounts
$ kubectl create -f etcd-operator-role.yaml
$ kubectl create -f etcd-operator-rolebinding.yaml
$ kubectl create -f etcd-operator-deployment.yaml
$ kubectl get deployments
$ kubectl get pods
$ kubectl create -f etcd-cluster-cr.yaml
$ kubectl get pods
$ kubectl describe etcdcluster/example-etcd-cluster
```

#### Exercises

```
$ kubectl get services --selector etcd_cluster=example-etcd-cluster
$ kubectl run --rm -i --tty etcdctl --image quay.io/coreos/etcd \
--restart=Never -- /bin/sh
$ export ETCDCTL_API=3
$ export ETCDCSVC=http://example-etcd-cluster-client:2379
$ etcdctl --endpoints $ETCDCSVC put foo bar
$ etcdctl --endpoints $ETCDCSVC get foo
foo
bar
$ kubectl delete pod example-etcd-cluster-95gqrthjbz
$ kubectl get pods -w
$ kubectl describe etcdcluster/example-etcd-cluster
$ kubectl run --rm -i --tty etcdctl --image quay.io/coreos/etcd \
--restart=Never -- /bin/sh
$ etcdctl --endpoints http://example-etcd-cluster-client:2379 cluster-health
$ kubectl get pod example-etcd-cluster-795649v9kq \
-o yaml | grep "image:" | uniq
image: quay.io/coreos/etcd:v3.1.10
$ kubectl patch etcdcluster example-etcd-cluster --type='json' \
-p '[{"op": "replace", "path": "/spec/version", "value":3.3.12}]'
$ kubectl describe etcdcluster/example-etcd-cluster
```

#### Cleaning up:
```
$ kubectl delete -f etcd-operator-sa.yaml
$ kubectl delete -f etcd-operator-role.yaml
$ kubectl delete -f etcd-operator-rolebinding.yaml
$ kubectl delete -f etcd-operator-crd.yaml
$ kubectl delete -f etcd-operator-deployment.yaml
$ kubectl delete -f etcd-cluster-cr.yaml
```
