Download cli tools from Supervisor API endpoint (https://LB-IP) - put them in a path'ed folder e.g /usr/local/bin

Authentication:
kubectl-vsphere login --server=10.101.7.11 --insecure-skip-tls-verify --vsphere-username=username@vsphere.local
Deploy a workload cluster in a vSphere Namespace:
´´´
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: psp:privileged
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - vmware-system-privileged
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: all:psp:privileged
roleRef:
  kind: ClusterRole
  name: psp:privileged
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  name: system:serviceaccounts
  apiGroup: rbac.authorization.k8s.io
´´´

