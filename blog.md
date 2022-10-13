# Quickly deploying ApplicationSets into the new global-namespace using RHACM 

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

apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  channel: "stable"
  installPlanApproval: Automatic
  name: openshift-gitops-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  config:
    env:
      - name: ARGOCD_CLUSTER_CONFIG_NAMESPACES
        value: 'openshift-gitops, open-cluster-management-global-set'

To allow for the global-namespace above to be managed by OpenShift GitOps, we have labeled them with the following:

oc label namespace open-cluster-management-global-set argocd.argoproj.io/managed-by=openshift-gitops

What we see above are scoped resources for the project created in GitOps. Normally, an ArgoCD admin creates a project and decides in advance which clusters and Git repositories it defines. This is not possible for ever changing teams and clusters. It creates a problem in scenarios where a developer wants to add a repository or cluster after the initial creation of the project. The above project definition shows a reliable way of offering a self service definition for allowing developers to add their own repositories and applications in the GitOps area.

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

Show PlacementRule field in PolicyGenerator
Show PlacementField in PolicyGenerator
Mention that Placement must exist



5. finally deploy the ApplicationSet and inspect it from ACM-UI and ArgoCD-UI

6. Deploy another ApplicationSet just for Policies and bootstrap the two ApplicationSets using AppOfApps Pattern
* If you deploy an ApplicationSet only to the Hub, you might also consider just using an ArgoCD-Application, but it's nice to show for demo-purposes.





