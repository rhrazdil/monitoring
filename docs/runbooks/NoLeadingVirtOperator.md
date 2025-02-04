# NoLeadingVirtOperator 

## Meaning

This alert is triggered when the system detects that in the past 10 minutes, there is no virt-operator pod holding the leader lease despite there are one or more virt-operator pod is in `Ready` state. The alert suggests that there is no operating virt-operator pod. 

## Impact

The virt-operator is the first k8s operator coming alive in a KubeVirt cluster. Its primary responsibilities are installation, live-update, live-upgrade of a KubeVirt cluster, monitoring the life-cycle of top-level controllers, such as virt-controller, virt-handler, virt-launcher, etc. and manage their reconciliation. In addition, virt-operator is responsible for cluster-wide tasks, such as certificate rotation and some infrastructure management, etc.  The virt-operator deployment has a default replica of 2 pods, with one leader pod holding a leader lease, indicating an operating virt-operator pod. 

This alert indicates a failure at the level of the KubeVirt cluster. Critical cluster-wide management functionalities such as certification rotation, KubeVirt upgrade, and reconciliation of KubeVirt controllers, may not be available for the time being.

## Diagnosis

The information on the leader status of a virt-operator pod can be deduced from the pod logs. The log messages containing `Started leading` and `acquire leader` should help deduce the leader status of a given virt-operator pod. 

In addition, always check whether there are any running virt-operator pods and their status, with following commands:
- `export NAMESPACE="$(kubectl get kubevirt -A -o custom-columns="":.metadata.namespace)"`
- `kubectl -n $NAMESPACE get pods -l kubevirt.io=virt-operator`
- `kubectl -n $NAMESPACE logs <pod-name>`.
- `kubectl -n $NAMESPACE describe pod <pod-name>`.


***Leader pod example:***
```
$ kubectl -n $NAMESPACE logs <pod-name> |grep lead
{"component":"virt-operator","level":"info","msg":"Attempting to acquire leader status","pos":"application.go:400","timestamp":"2021-11-30T12:15:18.635387Z"}
I1130 12:15:18.635452       1 leaderelection.go:243] attempting to acquire leader lease <namespace>/virt-operator...
I1130 12:15:19.216582       1 leaderelection.go:253] successfully acquired lease <namespace>/virt-operator
{"component":"virt-operator","level":"info","msg":"Started leading","pos":"application.go:385","timestamp":"2021-11-30T12:15:19.216836Z"}
```

***Non-leader pod example:***
```
$ kubectl -n $NAMESPACE logs <pod-name> |grep lead
{"component":"virt-operator","level":"info","msg":"Attempting to acquire leader status","pos":"application.go:400","timestamp":"2021-11-30T12:15:20.533696Z"}
I1130 12:15:20.533792       1 leaderelection.go:243] attempting to acquire leader lease <namespace>/virt-operator...
```

## Mitigation

There can be several reasons, try to identify the root cause and fix it. If you cannot fix it, please open an issue and attach the artifacts gathered in the Diagnosis section.