# Quickly deploying ApplicationSets into RHACM's new global-namespace  

Starting with  RHACM 2.6 we introduced a new global clusterset and in the following we would like to explain why this has been done, and use this new Global `ClusterSet` to quickly deploy on ApplicationSet
in the new global-namespace. As a side-information you will learn how to use Placement-Objects (and why/how you can migrate from PlacementRule-Objects) as well as how to deploy RHACM using Policy-Generator while not allowing PlacementRules to be deployed.

Multitenancy is a very important topic when working with a Multi-Cluster-Management solution. RHACM offers great concepts to achieve this but there are usecases where you want to have a shared view to the whole fleet of Clusters and for that reasons
we provided a new global-namespace, to give users also some kind of `Quick-Start`.

## What resources are provided out of the box

Starting with RHACM 2.6  there is a namespace called `open-cluster-management-global-set` and a `ManagedClusterSetBinding` called `global` to bind the global ManagedClusterSet to the `open-cluster-management-global-set` namespace. You can read [here](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html-single/multicluster_engine/index#managedclustersets_global)

See here how the `global` ClusterSet looks like. While you can only assign a Cluster to a single ClusterSet, each Cluster is also part of the Global-ClusterSet so in the below
example 9 Clusters (always all in the fleet) are assigned to it.

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
Let's take a look at the other objects:

* ManagedClusterSetBinding and 
* (Global)Namespace

The binding is a connection between the `ClusterSet` and `Global` Namespace

```
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
```

As mentioned all of the above resources are created by default, this already helps for starting quickly and easy when it comes to deploy applications.

## What resources still need to be created

* Gitops-Cluster-Resource

This resource provides a Connection between ArgoCD-Server and the Placement (where to deploy exactly the Application).  

```
---  
apiVersion: apps.open-cluster-management.io/v1beta1
kind: GitOpsCluster
metadata:
  name: global 
  namespace: open-cluster-management-global-set 
spec:
  argoServer:
    cluster: local-cluster
    argoNamespace: open-cluster-management-global-set 
  placementRef:
    kind: Placement
    apiVersion: cluster.open-cluster-management.io/v1beta1
    name: global-appset-placement 
    namespace: open-cluster-management-global-set 
```

* Placements

See here a Placement example which deploys to a fleet of Clusters, where the Clusters needs to be labeled with `useglobal=true`.

```
---
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: Placement
metadata:
  name: global-appset-placement 
  namespace: open-cluster-management-global-set 
spec:
  predicates:
  - requiredClusterSelector:
      labelSelector:
        matchExpressions:
          - {key: useglobal, operator: In, values: ["true"]}
```


## Configuration Options regarding Gitops-Operator


In the following we are discussing two configuration options and explain the differences:

ArgoCD can operate in two dfferent modes: `Namespace` and `Cluster`. For security reasons the namespaces to enable for Cluster mode can only be done in the subscription.
The `ARGOCD_CLUSTER_CONFIG_NAMESPACES` grants the specific Argo CD instance cluster-wider privileges (including setting up ClusterRoles), while the label approach only grants access to the labeled namespaces.


####  making `open-cluster-management-global-set` a Cluster-Scoped-Namespace for GitopsOperator

```
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
```
## Disabling Placement-Rules and Migrating Placement-Rules to Placement using PolicyGenerator

At one point in time in the future `PlacementRules` are going to be deprecated. Already now we want to disable them as we like to force the users to work with Placement objects. 
Note that ApplicationSets in ACM only work with Placement but there are some other artifacts like Subscriptions and Policies which still default to `PlacementRule` Object.

For disabling just set e.g. on the `AppProject` level in ArgoCD:

```
spec:
  clusterResourceBlacklist:
    - group: 'open-cluster-management.io'
      kind: PlacementRule
```

* Migrate from `PlacementRules` to `Placement` using `PolicyGenerator`

As mentioned we also want to deploy RHACM-Policies. For this we are using PolicyGenerator. See here it's
[reference file](https://github.com/stolostron/policy-generator-plugin/blob/main/docs/policygenerator-reference.yaml)

You see in above file that there is both the option to set a PlacementRule or a Placement.
You can either specify a name (when the object already exists in the Cluster) or a path in the Gitrepo
to apply the objects. See: `placementPath`,`placementName` or `placementRulePath` and `placementRuleName`.


This is more or less all you need to know for migrating between the two objects.

## finally deploy the ApplicationSet and inspect it from ACM-UI and ArgoCD-UI


## Deploy another ApplicationSet just for Policies and bootstrap the two ApplicationSets using AppOfApps Pattern

* If you deploy an ApplicationSet only to the Hub, you might also consider just using an ArgoCD-Application, but it's nice to show for demo-purposes.


Let's verify that you cannot process PlacementRules via Git.

## Closing words:

To summarize we wanted to show you how quickly and easy you can deploy Applications using RHACM to a fleet of Clusters.  Additionally we wanted to explain some additional concepts highlighting
the evolution of the Product (e.g PlacementRule-->Placement).




