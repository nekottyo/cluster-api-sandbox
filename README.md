cluster-api-sandbox
---

EKS

```console

❯ clusterctl version
clusterctl version: &version.Info{Major:"1", Minor:"0", GitVersion:"v1.0.0", GitCommit:"e09ed61cc9ba8bd37b0760291c833b4da744a985", GitTreeState:"clean", BuildDate:"", GoVersion:"go1.17.1", Compiler:"gc", Platform:"darwin/amd64"}

❯ clusterawsadm version
clusterawsadm version: &version.Info{Major:"1", Minor:"0", GitVersion:"v1.0.0", GitCommit:"0fb108ac19f544a33a7c31a5278f212fbb88ccc5", GitTreeState:"clean", BuildDate:"2021-10-08T13:56:58Z", GoVersion:"go1.16.8", AwsSdkVersion:"v1.40.56", Compiler:"gc", Platform:"darwin/amd64"}
```

```console
❯ clusterawsadm bootstrap iam create-cloudformation-stack
Attempting to create AWS CloudFormation stack cluster-api-provider-aws-sigs-k8s-io

Following resources are in the stack:

Resource                  |Type                                                                                  |Status
AWS::IAM::InstanceProfile |control-plane.cluster-api-provider-aws.sigs.k8s.io                                    |CREATE_COMPLETE
AWS::IAM::InstanceProfile |controllers.cluster-api-provider-aws.sigs.k8s.io                                      |CREATE_COMPLETE
AWS::IAM::InstanceProfile |nodes.cluster-api-provider-aws.sigs.k8s.io                                            |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::0000000000:policy/control-plane.cluster-api-provider-aws.sigs.k8s.io   |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::0000000000:policy/nodes.cluster-api-provider-aws.sigs.k8s.io           |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::0000000000:policy/controllers.cluster-api-provider-aws.sigs.k8s.io     |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::0000000000:policy/controllers-eks.cluster-api-provider-aws.sigs.k8s.io |CREATE_COMPLETE
AWS::IAM::Role            |control-plane.cluster-api-provider-aws.sigs.k8s.io                                    |CREATE_COMPLETE
AWS::IAM::Role            |controllers.cluster-api-provider-aws.sigs.k8s.io                                      |CREATE_COMPLETE
AWS::IAM::Role            |eks-controlplane.cluster-api-provider-aws.sigs.k8s.io                                 |CREATE_COMPLETE
AWS::IAM::Role            |nodes.cluster-api-provider-aws.sigs.k8s.io                                            |CREATE_COMPLETE
```



```console
❯ export AWS_B64ENCODED_CREDENTIALS=$(clusterawsadm bootstrap credentials encode-as-profile)
```

```console
❯ clusterctl init --infrastructure aws

Fetching providers
Installing cert-manager Version="v1.5.3"
Waiting for cert-manager to be available...
Installing Provider="cluster-api" Version="v1.0.0" TargetNamespace="capi-system"
Installing Provider="bootstrap-kubeadm" Version="v1.0.0" TargetNamespace="capi-kubeadm-bootstrap-system"
Installing Provider="control-plane-kubeadm" Version="v1.0.0" TargetNamespace="capi-kubeadm-control-plane-system"
Installing Provider="infrastructure-aws" Version="v1.0.0" TargetNamespace="capa-system"
I1021 12:53:09.550812   97861 request.go:665] Waited for 1.003186709s due to client-side throttling, not priority and fairness, request: GET:https://127.0.0.1:49682/apis/cluster.x-k8s.io/v1alpha4?timeout=30s

Your management cluster has been initialized successfully!

You can now create your first workload cluster by running the following:

  clusterctl generate cluster [name] --kubernetes-version [version] | kubectl apply -f -


❯ k get po -A
NAMESPACE                           NAME                                                             READY   STATUS              RESTARTS   AGE
capa-system                         capa-controller-manager-869875c879-mqbkb                         0/1     ContainerCreating   0          16s
capi-kubeadm-bootstrap-system       capi-kubeadm-bootstrap-controller-manager-fb4766f64-7rfqx        0/1     ContainerCreating   0          39s
capi-kubeadm-control-plane-system   capi-kubeadm-control-plane-controller-manager-86b5f554dd-mfjnl   0/1     ContainerCreating   0          32s
capi-system                         capi-controller-manager-8f66f5b7b-j78v6                          0/1     Running             0          46s
cert-manager                        cert-manager-848f547974-mjmxd                                    1/1     Running             0          82s
cert-manager                        cert-manager-cainjector-54f4cc6b5-k5l9m                          1/1     Running             0          82s
cert-manager                        cert-manager-webhook-7c9588c76-9t52c                             1/1     Running             0          82s
kube-system                         coredns-558bd4d5db-jbnmh                                         1/1     Running             0          9m22s
kube-system                         coredns-558bd4d5db-wm9jw                                         1/1     Running             0          9m22s
kube-system                         etcd-kind-control-plane                                          1/1     Running             0          9m38s
kube-system                         kindnet-kd7nm                                                    1/1     Running             0          9m22s
kube-system                         kube-apiserver-kind-control-plane                                1/1     Running             0          9m26s
kube-system                         kube-controller-manager-kind-control-plane                       1/1     Running             0          9m39s
kube-system                         kube-proxy-md9vl                                                 1/1     Running             0          9m22s
kube-system                         kube-scheduler-kind-control-plane                                1/1     Running             0          9m26s
local-path-storage                  local-path-provisioner-547f784dff-zmx5z                          1/1     Running             0          9m22s
```

```console
❯ clusterctl generate cluster capi-eks-quickstart --flavor eks --kubernetes-version v1.21.0 --worker-machine-count=3 > capi-eks-quickstart.yaml
```


```console
❯ clusterctl move --to-kubeconfig=capi.conf --dry-run
Performing move...
********************************************************
This is a dry-run move, will not perform any real action
********************************************************
Discovering Cluster API objects
Moving Cluster API objects Clusters=1
Creating objects in the target cluster
Deleting objects from the source cluster
```

```console
❯ k config current-context
arn:aws:eks:ap-northeast-1:0000000000:cluster/default_capi-eks-quickstart-control-plane

❯ k get cluster
NAME                  PHASE          AGE   VERSION
capi-eks-quickstart   Provisioning   61s
```

```console
❯ clusterctl describe cluster capi-eks-quickstart
NAME                                                                       READY  SEVERITY  REASON  SINCE  MESSAGE
/capi-eks-quickstart                                                       True                     78s
├─ControlPlane - AWSManagedControlPlane/capi-eks-quickstart-control-plane  True                     76s
└─Workers
  └─MachineDeployment/capi-eks-quickstart-md-0                             True                     18m
    └─3 Machines...                                                        True                     77s    See capi-eks-quickstart-md-0-85885464cc-chhng, capi-eks-quickstart-md-0-85885464cc-cmqtv, ...
```
