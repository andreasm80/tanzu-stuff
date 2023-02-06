Download cli tools from Supervisor API endpoint (https://LB-IP) - put them in a path'ed folder e.g /usr/local/bin

Authentication:
```
kubectl-vsphere login --server=10.101.7.11 --insecure-skip-tls-verify --vsphere-username=username@vsphere.local
```
Deploy a workload cluster in a vSphere Namespace:

Example yaml for vSphere 7 (without proxy and certs, just uncomment if needed:
```
apiVersion: run.tanzu.vmware.com/v1alpha2
kind: TanzuKubernetesCluster
metadata:
  name: tkc-cluster-1
  namespace: dc-ns-1
spec:
  topology:
    controlPlane:
      replicas: 3
      vmClass: best-effort-medium
      storageClass: tanzu-default-storage
      tkr:  
        reference:
          name: v1.23.8---vmware.3-tkg.1
    nodePools:
    - name: pool-1
      replicas: 3
      vmClass: best-effort-medium
      storageClass: tanzu-default-storage
      tkr:  
        reference:
          name: v1.23.8---vmware.3-tkg.1
  settings:
    storage:
      defaultClass: tanzu-default-storage
    network:
      cni:
        name: antrea
      services:
        cidrBlocks: ["10.96.0.0/12"]
      pods:
        cidrBlocks: ["192.168.0.0/16"]
      serviceDomain: cluster.local
#      proxy:
#        httpProxy: http://proxy.server.com:8080
#        httpsProxy: http://proxy.server.com:8080
#        noProxy: [10.96.0.0/12,192.168.0.0/16,"mgmt-frontend-workload-network-cidr",.local,.svc,.svc.cluster.local]
#      trust:
#        additionalTrustedCAs:
#          - name: CompanyInternalCA-1
#            data: LS0tLS1C...LS0tCg==
#          - name: CompanyInternalCA-2
#            data: MTLtMT1C...MT0tPg==
```

Workload cluster example for vSphere 8:
```
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: wdc-2-tkc-cluster-1
  namespace: wdc-2-ns-1
spec:
  clusterNetwork:
    services:
      cidrBlocks: ["20.10.0.0/16"]
    pods:
      cidrBlocks: ["20.20.0.0/16"]
    serviceDomain: "cluster.local"
  topology:
    class: tanzukubernetescluster
    version: v1.23.8---vmware.2-tkg.2-zshippable
    controlPlane:
      replicas: 1
      metadata:
        annotations:
          run.tanzu.vmware.com/resolve-os-image: os-name=ubuntu
    workers:
      machineDeployments:
        - class: node-pool
          name: node-pool-01
          replicas: 3
          metadata:
            annotations:
              run.tanzu.vmware.com/resolve-os-image: os-name=ubuntu
    variables:
      - name: vmClass
        value: best-effort-medium
      - name: storageClass
        value: vsan-default-storage-policy
```

AntreaCOnfig example to apply on the namespace:
```
apiVersion: cni.tanzu.vmware.com/v1alpha1
kind: AntreaConfig
metadata:
  name: wdc-2-tkc-cluster-1-antrea-package
  namespace: wdc-2-ns-1
spec:
  antrea:
    config:
      featureGates:
        AntreaProxy: true
        EndpointSlice: false
        AntreaPolicy: true
        FlowExporter: false
        Egress: true
        NodePortLocal: true
        AntreaTraceflow: true
        NetworkPolicyStats: true
```




PSP ClusterRole
```
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
```

Auhtentication to ns and ns and tkc cluster:
```
kubectl-vsphere login --server=10.102.7.11 --insecure-skip-tls-verify --vsphere-username=supervisor-manager@cpod-nsxam-wdc.az-wdc.cloud-garage.net --tanzu-kubernetes-cluster-namespace wdc-2-ns-1
```
```
kubectl-vsphere login --server=10.102.7.11 --insecure-skip-tls-verify --vsphere-username=supervisor-manager@cpod-nsxam-wdc.az-wdc.cloud-garage.net --tanzu-kubernetes-cluster-namespace wdc-2-ns-1 --tanzu-kubernetes-cluster-name=wdc-2-tkc-cluster-1
```
