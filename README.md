# blogapplicationsetsglobalnamespace

Starting with  RHACM 2.6 we introduced a new global clusterset

In the following we would like to explain why this has been done, and use this new GlobalSet to quickly deploy on ApplicationSet
in the global-namespace 

In the following you will learn:

1. What resources are provided out of the box

* Global-ClusterSet
* GlobalBinding
* GlobalNamespace

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
