# ACM and ArgoCD integration with application deployment

ACM and ArgoCD integration that deploys an application to all ACM managed cluster using ArgoCD.

### Idea

The idea is to have ACM managing clusters and give Argo visibility to it so Argo can sync your application repositories with them.

### Prerequisites

We will need at least *two* fresh installed OCP clusters, one to act as Hub and another to be added as a managed cluster so we can see Argo sync and deploy the resources onto it.

### Install and execution

First lets deploy ACM Hub operator and OpenShift GitOps operator which will also create your ACM Hub and ArgoCD instances.

```
until oc apply -k gitops/manifests/bootstrap/openshift-gitops-operator/base; do sleep 5; done
```

```
until oc apply -k gitops/manifests/bootstrap/advanced-cluster-management/base; do sleep 15; done
```
In the ACM view in the console, we will add the spoke cluster as managed cluster, this is a manual step, we will have to add labels to the managed and local clusters so the integration between ACM and Argo works (via ManagedClusterSet).

When adding the remote cluster, note that it must be in the `demo` cluster set. Also make sure your `local-cluster` is transfered to the `demo` cluster set.

Your `local-cluster` needs the `env=test` label and your remote cluster needs the `env=prod` label.

As a option to the UI method of assigning the clusters to the `demo` cluster set and adding the proper labels you can execute this:

```
oc label managedcluster <name-of-your-remote-cluster> cluster.open-cluster-management.io/clusterset=demo env=prod
oc label managedcluster local-cluster cluster.open-cluster-management.io/clusterset=demo env=test
```

The last step is to create the `ApplicationSet` in Argo that will create application objects to represent every deployment on every cluster that are part of the cluster set `demo`. We can create it by executing: 

```
oc apply -k gitops/manifests/content/demo/apps/base
```

### Validation  

After provisioning the operators and deploying the application using the app of apps pattern with Argo, we can validate it by checking:

Your managed clusters:

```
oc get managedcluster
```

Your placement rule:

```
oc get placement -n openshift-gitops
```

Your GitOps cluster:

```
oc get gitopscluster demo-gitops-cluster -n openshift-gitops -o=jsonpath='{.status.message}'
```

Now, open the Argo web console and check the two created applications and that is synced with both clusters, also check the pods under the `welcome` project, they should be running the nodejs sample application.