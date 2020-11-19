### Application Overview

• A web frontend, implemented in React

• A REST API, implemented in Python using the Django framework

• A database, using MySQL

<img src="https://github.com/adavarski/k8s-operators-playground/blob/main/screenshots/app.png?raw=true" width="650">


### Build docker images

```
$ docker login

$ cd docker/visitor-service
$ docker build --tag visitors-service:1.0.0 . 
$ docker push davarski/visitors-service:1.0.0

$ cd docker/visitor-webui
$ docker build --tag visitors-webui:1.0.0 .
$ docker push davarski/visitors-webui:1.0.0
```

### Setup environment 
```
$ ./setup_environment.sh
```

Check:

```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:18:23Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
```

### Manifest-based installation

```
$ cd manifests
$ kubectl apply -f database.yaml
$ kubectl apply -f backend.yaml
$ kubectl apply -f frontend.yaml
$ minikube ip
192.168.99.100
```

You can access the Visitors Site by opening a browser and
going to http://192.168.99.100:30686.

<img src="https://github.com/adavarski/k8s-operators-playground/blob/main/screenshots/visitors-dashboard.png?raw=true" width="650">


Cleaning up:

```
$ kubectl delete -f frontend.yaml
$ kubectl delete -f backend.yaml
$ kubectl delete -f database.yaml
```
### k8s operator-based installation etcd: demo

Deploy:

```
$ cd k8s-operator-etcd-demo
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

Exercices:

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

Cleaning up:
```
$ kubectl delete -f etcd-operator-sa.yaml
$ kubectl delete -f etcd-operator-role.yaml
$ kubectl delete -f etcd-operator-rolebinding.yaml
$ kubectl delete -f etcd-operator-crd.yaml
$ kubectl delete -f etcd-operator-deployment.yaml
$ kubectl delete -f etcd-cluster-cr.yaml
```

### Adapter Operators

<img src="https://github.com/adavarski/k8s-operators-playground/blob/main/screenshots/k8s-operators-model.png?raw=true" width="650">


#### Helm Operator

Building the Operator (or use configured k8s-operators/visitors-helm/visitors-helm-ready)

```
$ cd k8s-operators/visitors-helm/helm  
$ OPERATOR_NAME=visitors-helm-operator
$ operator-sdk new $OPERATOR_NAME --api-version=example.com/v1 \
--kind=VisitorsApp --type=helm
```

Running the Helm Operator outside of the cluster:

```
$ cp watches.yaml local-watches.yaml
$ kubectl apply -f deploy/crds/*_crd.yaml
$ operator-sdk up local --watches-file ./local-watches.yaml
INFO[0000] Running the operator locally.
INFO[0000] Using namespace default.
```

Running an Operator as a Deployment Inside a Cluster:

```
$ operator-sdk build davarski/visitors-helm-operator:0.1
```
Update the deploy/operator.yaml file that the SDK gen‐
erates with the name of the image. The field to update is named image and can be
found under:
spec -> template -> spec -> containers

```
$ kubectl apply -f deploy/crds/*_crd.yaml
$ kubectl apply -f deploy/service_account.yaml
$ kubectl apply -f deploy/role.yaml
$ kubectl apply -f deploy/role_binding.yaml
$ kubectl apply -f deploy/operator.yaml
```
Cleaning up:

```
$ kubectl delete -f deploy/operator.yaml
$ kubectl delete -f deploy/role_binding.yaml
$ kubectl delete -f deploy/role.yaml
$ kubectl delete -f deploy/service_account.yaml
$ kubectl delete -f deploy/crds/*_crd.yaml
```
#### Ansible Operator

Building the Operator (or use configured k8s-operators/visitors-ansible/visitors-ansible-ready):

```
$ cd k8s-operators/visitors-ansible/ansible
$ OPERATOR_NAME=visitors-ansible-operator
$ operator-sdk new $OPERATOR_NAME --api-version=example.com/v1 \
--kind=VisitorsApp --type=ansible
```
Running the Ansible Operator outside of the cluster:
```
$ cp watches.yaml local-watches.yaml
$ kubectl apply -f deploy/crds/*_crd.yaml
$ operator-sdk up local --watches-file ./local-watches.yaml
INFO[0000] Running the operator locally.
INFO[0000] Using namespace default.
```

Running an Operator as a Deployment Inside a Cluster:

```
$ operator-sdk build davarski/visitors-ansible-operator:0.1
```
Update the deploy/operator.yaml file that the SDK gen‐
erates with the name of the image. The field to update is named image and can be
found under:
spec -> template -> spec -> containers

```
$ kubectl apply -f deploy/crds/*_crd.yaml
$ kubectl apply -f deploy/service_account.yaml
$ kubectl apply -f deploy/role.yaml
$ kubectl apply -f deploy/role_binding.yaml
$ kubectl apply -f deploy/operator.yaml
```
Cleaning up:
```
$ kubectl delete -f deploy/operator.yaml
$ kubectl delete -f deploy/role_binding.yaml
$ kubectl delete -f deploy/role.yaml
$ kubectl delete -f deploy/service_account.yaml
$ kubectl delete -f deploy/crds/*_crd.yaml
```

#### Go Operator

Building the Operator:

Since the Operator is written in Go, the project skeleton must adhere to the language
conventions. In particular, the Operator code must be located in your $GOPATH . See
the GOPATH documentation for more information.

```
$ cd k8s-operators/visitors-ansible
$ OPERATOR_NAME=visitors-go-operator
$ operator-sdk new $OPERATOR_NAME
```

```
The Operator SDK generated many of the files in that repository. The files that were
modified to run the Visitors Site application are:
deploy/crds/
• example_v1_visitorsapp_crd.yaml
— This file contains the CRD.
• example_v1_visitorsapp_cr.yaml
— This file defines a CR with sensible example data.
pkg/apis/example/v1/visitorsapp_types.go
• This file contains Go objects that represent the CR, including its spec and status
fields.
pkg/controller/visitorsapp/
• backend.go, frontend.go, mysql.go
— These files contain all of the information specific to deploying those compo‐
nents of the Visitors Site. This includes the deployments and services that the
Operator maintains, as well as the logic to handle updating existing resources
when the end user changes the CR.
• common.go
— This file contains utility methods used to ensure the deployments and serv‐
ices are running, creating them if necessary.
• visitorsapp_controller.go
— The Operator SDK initially generated this file, which was then modified for
the Visitors Site–specific logic. The Reconcile method contains the majority
of the changes; it drives the overall flow of the Operator by calling out to func‐
tions in the previously described files.

```
```
$ operator-sdk add api --api-version=example.com/v1 --kind=VisitorsApp
$ operator-sdk add controller --api-version=example.com/v1 --kind=VisitorsApp
```
Running the Go Operator outside of the cluster:

```
$ kubectl apply -f deploy/crds/*_crd.yaml
$ export OPERATOR_NAME=xxxxx
$ operator-sdk up local --namespace default
$ kubectl apply -f deploy/crds/*_cr.yaml
```

Running an Operator as a Deployment Inside a Cluster:

```
$ operator-sdk build davarski/visitors-go-operator:0.1
```
Update the deploy/operator.yaml file that the SDK gen‐
erates with the name of the image. The field to update is named image and can be
found under:
spec -> template -> spec -> containers

```
$ kubectl apply -f deploy/crds/*_crd.yaml
$ kubectl apply -f deploy/service_account.yaml
$ kubectl apply -f deploy/role.yaml
$ kubectl apply -f deploy/role_binding.yaml
$ kubectl apply -f deploy/operator.yaml
```
Cleaning up:

```
$ kubectl delete -f deploy/operator.yaml
$ kubectl delete -f deploy/role_binding.yaml
$ kubectl delete -f deploy/role.yaml
$ kubectl delete -f deploy/service_account.yaml
$ kubectl delete -f deploy/crds/*_crd.yaml
```
### Operator Lifecycle Manager (OLM)

Check OLM:

```
$ kubectl get ns olm
$ kubectl get pods -n olm
$ kubectl get crd
$ kubectl get catalogsource -n olm
$ kubectl describe catalogsource/operatorhubio-catalog -n olm
$ kubectl get packagemanifest -n olm
```
Demo (etcd k8s operator):

```
$ cd OLM/demo-etcd
$ kubectl apply -f all-og.yaml
$ kubectl describe packagemanifest/etcd -n olm
$ kubectl apply -f sub.yaml
$ kubectl get csv -n default
$ kubectl describe csv/etcdoperator.v0.9.4 -n default
$ kubectl get deployment -n default
$ kubectl get deployment/etcd-operator -n default -o yaml
$ kubectl get pods -n default
$ echo "Deleting the Operator"; kubectl delete csv/etcdoperator.v0.9.4
```
OLM Bundle Creation

```
$ cd k8s-operators/visitors-ansible/visitors-ansible-ready/visitors-ansible-operator
$ operator-sdk olm-catalog gen-csv --csv-version 1.0.0
$ cd deploy/olm-catalog/visitors-ansible-operator
$ mkdir bundle; cp 1.0.0/visitors-ansible-operator.v1.0.0.clusterserviceversion.yaml bundle;  cp visitors-ansible-operator.package.yaml bundle
$ cp ../../crds/example.com_visitorsapps_crd.yaml bundle/example.com_v1_visitorsapps_crd.yaml 
```
Edit bundle files:

```
bundle/visitors-ansible-operator.v1.0.0.clusterserviceversion.yaml

      displayName: Visitors Site Application
      description: Full stack for the Visitors Site
      resources:
      - kind: Service
        version: v1
      - kind: Deployment
        version: v1
      specDescriptors:
      - displayName: Size
        description: The number of backend pods to create
        path: size
      - displayName: Title
        description: Title to display on the web UI dashboard
        path: title
      statusDescriptors:
      - displayName: Backend Image
        description: Full name and version of the image used to create the backend service
        path: backendImage
      - displayName: Frontend Image
        description: Full name and version of the image used to create the frontend service
        path: frontendImage
  description: Operator for installing the Visitors Site application
  displayName: Visitors Ansible Operator

bundle/example_v1_visitorsapp_crd.yaml 

  validation:
    openAPIV3Schema:
      properties:
        apiVersion:
          description: 'APIVersion defines the versioned schema of this representation
            of an object. Servers should convert recognized schemas to the latest
            internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/api-conventions.md#resources'
          type: string
        kind:
          description: 'Kind is a string value representing the REST resource this
            object represents. Servers may infer this from the endpoint the client
            submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds'
          type: string
        metadata:
          type: object
        spec:
          type: object
          properties:
            size:
              type: integer
            title:
              type: string
          required:
          - size
        status:
          type: object
          properties:
            backendImage:
              type: string
            frontendImage:
              type: string
```
```
$ cp -a bundle/ k8s-operators-playground/OLM/ 
$ cd k8s-operators-playground/OLM/
```

Create the OperatorSource:

```
$ cd OLM
$ kubectl apply -f operator-source.yaml
$ kubectl get catalogsource -n marketplace
$ kubectl get opsrc -n marketplace
```
If there are no bundles at the endpoint when you create the source, the status will
be Failed . You can ignore this for now; you’ll refresh this list later, once you’ve
uploaded a bundle.

Building the OLM Bundle:
```
$ operator-courier verify ./bundle
$ ./get-quay-token 
Username: davarski
Password: 

========================================
Auth Token is: basic XXXXXXXXXX=
The following command will assign the token to the expected variable:
  export QUAY_TOKEN="basic XXXXXXXXX="
$ OPERATOR_DIR=bundle
$ QUAY_NAMESPACE=davarski
PACKAGE_NAME=visitors-ansible-operator
PACKAGE_VERSION=1.0.0
QUAY_TOKEN="basic XXXXXXXXX="
$ operator-courier push "$OPERATOR_DIR" "$QUAY_NAMESPACE" \
"$PACKAGE_NAME" "$PACKAGE_VERSION" "$QUAY_TOKEN"

```
By default, bundles pushed to Quay.io in this fashion are marked as
private. Navigate to the image at https://quay.io/application/ and
mark it as public so that it is accessible to the cluster.

<img src="https://github.com/adavarski/k8s-operators-playground/blob/main/screenshots/quay.io-applications.png?raw=true" width="650">


```
$ kubectl get pods -n marketplace 
$ kubectl delete pod davarski-operators-5969c68d68-vfff6 -n marketplace
$ kubectl delete -f operator-source.yaml
$ kubectl apply -f operator-source.yaml

$ OP_SRC_NAME=davarski-operators
$ kubectl get opsrc $OP_SRC_NAME -o=custom-columns=NAME:.metadata.name,PACKAGES:.status.packages -n marketplace
NAME                 PACKAGES
davarski-operators   visitors-ansible-operator
```

Installing the Operator Through OLM
```
$ kubectl apply -f operator-group.yaml
$ kubectl apply -f subscription.yaml
$ kubectl get pods -n marketplace
```

Cleaning up:
```
$ kubectl delete -f subscription.yaml
$ kubectl delete csv/visitors-ansible-operator
...
```

