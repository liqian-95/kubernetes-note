### kubernetes常用命令

-   查看命令——kubectl get
```
Display one or many resources
Valid resource types include:

  * all
  * certificatesigningrequests (aka 'csr')
  * clusterrolebindings
  * clusterroles
  * clusters (valid only for federation apiservers)
  * componentstatuses (aka 'cs')
  * configmaps (aka 'cm')
  * controllerrevisions
  * cronjobs
  * customresourcedefinition (aka 'crd')
  * daemonsets (aka 'ds')
  * deployments (aka 'deploy')
  * endpoints (aka 'ep')
  * events (aka 'ev')
  * horizontalpodautoscalers (aka 'hpa')
  * ingresses (aka 'ing')
  * jobs
  * limitranges (aka 'limits')
  * namespaces (aka 'ns')
  * networkpolicies (aka 'netpol')
  * nodes (aka 'no')
  * persistentvolumeclaims (aka 'pvc')
  * persistentvolumes (aka 'pv')
  * poddisruptionbudgets (aka 'pdb')
  * podpreset
  * pods (aka 'po')
  * podsecuritypolicies (aka 'psp')
  * podtemplates
  * replicasets (aka 'rs')
  * replicationcontrollers (aka 'rc')
  * resourcequotas (aka 'quota')
  * rolebindings
  * roles
  * secrets
  * serviceaccounts (aka 'sa')
  * services (aka 'svc')
  * statefulsets
  * storageclasses

#查看所有的namespace
$ kubectl get namespace
# 查看节点
$ kubectl get nodes
#查看namespace所有信息
$ kubectl get --namespace=xxx all
#查看namespace其他信息，如replicationcontrollers
$ kubectl get --namespace=xxx replicationcontrollers
```
-   label
```
Update the labels on a resource
Examples:
  # Update pod 'foo' with the label 'unhealthy' and the value 'true'.
  kubectl label pods foo unhealthy=true

  # Update pod 'foo' with the label 'status' and the value 'unhealthy', overwriting any existing value.
  kubectl label --overwrite pods foo status=unhealthy

  # Update all pods in the namespace
  kubectl label pods --all status=unhealthy

  # Update a pod identified by the type and name in "pod.json"
  kubectl label -f pod.json status=unhealthy

  # Update pod 'foo' only if the resource is unchanged from version 1.
  kubectl label pods foo status=unhealthy --resource-version=1

  # Update pod 'foo' by removing a label named 'bar' if it exists.
  # Does not require the --overwrite flag.
  kubectl label pods foo bar-
Usage:
  kubectl label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version] [options]

创建标签
$ kubectl label node k8s-node1 node1=test1
查看标签
$ kubectl get node --show-labels
```
-   更新、修改-rolling-update
此命令更新一个已有replicationController的信息
```
#修改namespace=nginx-server 其中的一个副本控制器nginx-mysql 里的所有镜像改为langtodu/nginx-php:v2.3，且副本镜像控制器命名为nginx-mysql-
$ kubectl rolling-update --namespace=nginx-server nginx-mysql nginx-mysql-v1 --image=langtodu/nginx-php:v2.3
```
-   rollout
```
Manage the rollout of a resource
Valid resource types include:

  * deployments
  * daemonsets
  * statefulsets
usage: kubectl rollout SUBCOMMAND [options]
Available Commands:
  history     View rollout history
  pause       Mark the provided resource as paused
  resume      Resume a paused resource
  status      Show the status of the rollout
  undo        Undo a previous rollout
Examples:
  # Rollback to the previous deployment
  kubectl rollout undo deployment/abc

  # Check the rollout status of a daemonset
  kubectl rollout status daemonset/foo
```
-   scale
```
Set a new size for a Deployment, ReplicaSet, Replication Controller, or Job.
Examples:
  # Scale a replicaset named 'foo' to 3.
  kubectl scale --replicas=3 rs/foo

  # Scale a resource identified by type and name specified in "foo.yaml" to 3.
  kubectl scale --replicas=3 -f foo.yaml

  # If the deployment named mysql's current size is 2, scale mysql to 3.
  kubectl scale --current-replicas=2 --replicas=3 deployment/mysql

  # Scale multiple replication controllers.
  kubectl scale --replicas=5 rc/foo rc/bar rc/baz

  # Scale job named 'cron' to 3.
  kubectl scale --replicas=3 job/cron
kubectl scale [--resource-version=version] [--current-replicas=count] --replicas=COUNT (-f FILENAME | TYPE NAME) [options]
```
