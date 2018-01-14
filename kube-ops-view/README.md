# OpenShift Demo kube ops view
Kubernetes Operational View Demo in OpenShift. Watch your OpenShift Cluster capacity with kube ops view.

# Getting Started

## Prerequisites
Running OpenShift Cluster :-)

## Installing

```
oc new-project ocp-ops-view
oc create sa kube-ops-view
oc adm policy add-scc-to-user anyuid -z kube-ops-view
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:ocp-ops-view:kube-ops-view
oc apply -f https://raw.githubusercontent.com/raffaelespazzoli/kube-ops-view/ocp/deploy-openshift/kube-ops-view.yaml
oc expose svc kube-ops-view
oc get route | grep kube-ops-view | awk '{print $2}'
```

# Authors
Dirk Hessenmüller

# License
Tis project is licensed under the MIT License.

# Acknowledgments

# References
https://blog.openshift.com/full-cluster-capacity-management-monitoring-openshift/
https://github.com/hjacobs/kube-ops-view


