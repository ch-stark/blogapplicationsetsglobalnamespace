# blogapplicationsetsglobalnamespace

Starting with  RHACM 2.6 we introduced a new global clusterset

In the following we would like to explain why this has been done, and use this new GlobalSet to quickly deploy on ApplicationSet
in the global-namespace 

In the following you will learn:

1. What resources are provided out of the box

starting with RHACM there is a namespace called open-cluster-management-global-set and a ManagedClusterSetBinding called global to bind the global ManagedClusterSet to the open-cluster-management-global-set namespace. You can read here
https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html-single/multicluster_engine/index#managedclustersets_global

* Global-ClusterSet
```
oc get ManagedClusterSet global -oyaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: ManagedClusterSet
metadata:
  name: global
spec:
  clusterSelector:
    labelSelector: {}
    selectorType: LabelSelector
status:
  conditions:
  - lastTransitionTime: "2022-09-13T18:49:31Z"
    message: 9 ManagedClusters selected
    reason: ClustersSelected
    status: "False"
    type: ClusterSetEmpty

```


While you can only assign a Cluster to a single ClusterSet, each Cluster is also part of the Global-ClusterSet

* GlobalBinding and * GlobalNamespace

The binding is a connection between ClusterSet and GlobalNamespace

oc get ManagedClusterSetBinding global -n open-cluster-management-global-set -oyaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: ManagedClusterSetBinding
metadata:
  name: global
  namespace: open-cluster-management-global-set
spec:
  clusterSet: global
status:
  conditions:
  - lastTransitionTime: "2022-09-13T18:49:32Z"
    message: ""
    reason: ClusterSetBound
    status: "True"
    type: Bound


2. What resources still need to be created

* Gitops-Cluster-Resource
* Placements

3. tuning options Gitops-Operator (cluster-scroped namespace versus Managed-by)



4. Disabling Placement-Rules and Migrating Placement-Rules to Placement using PolicyGenerator

* Disabling PlacementRules

It has been disabled also on the AppProject level in ArgoCD:

spec:
  namespaceResourceBlacklist:
    - group: 'open-cluster-management.io'
      kind: PlacementRule
  destinations:
    - namespace: blueteam-*
      server: '*'


* Migrate from PlacementRules to Placement using PolicyGenerator

5. finally deploy the ApplicationSet and inspect it from ACM-UI and ArgoCD-UI

6. Deploy another ApplicationSet just for Policies and bootstrap the two ApplicationSets using AppOfApps Pattern
* If you deploy an ApplicationSet only to the Hub, you might also consider just using an ArgoCD-Application, but it's nice to show for demo-purposes.





