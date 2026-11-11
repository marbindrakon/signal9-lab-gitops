# cluster-builds

One Argo CD `Application` (`<cluster>-deploy`) per spoke cluster, written here
by the `gitops_cluster_build` Ansible role (`ansible/roles/gitops_cluster_build`,
template `cluster-deploy-app.yaml.j2`) when the AAP deploy job runs. Each
renders `charts/mce-cluster-build` (or `charts/hcp-cluster-build`) from this
branch with the cluster's
`argocd/clusters/managed-clusters/<cluster>/helm-values/build-values.yaml`.

Do not add files here by hand. The directory is synced by the `acm-hub-site`
ApplicationSet (`../../site-apps.yaml`) into namespace `openshift-gitops`.
Standalone spokes are then configured by AutoShift from their entry in
`autoshift/managed-clusters.yaml`.

## Removing a cluster

Removing a file here does **not** destroy a cluster. The ApplicationSet syncs
without prune, so the `<cluster>-deploy` Application (and the cluster) stays.
Destroying a cluster is an explicit action of the AAP undeploy job
(`ansible/playbooks/undeploy_cluster.yml`, `_cluster_teardown.yml`):

1. it deletes the `<cluster>-deploy` Application on the hub (foreground). Argo's
   `resources-finalizer.argocd.argoproj.io` runs the chart's PreDelete hook,
   which deletes the Hive ClusterDeployment and waits for the deprovision, then
   cascade-deletes the rest of the chart;
2. it waits until the Application, the ClusterDeployment and the ManagedCluster
   are gone (default 11100 s, the chart's `deprovision.timeoutSeconds` plus a
   margin) and fails if they are not;
3. only then does it remove this file and the cluster's AutoShift entry from
   Git, the load balancer config, the Vault auth mount, SSO client, Ceph
   config, allocated resources and DNS records.

Protected clusters are rendered without the finalizer, and the plain undeploy
refuses to delete such an Application. `undeploy_protected_cluster.yml` first
re-renders it with the finalizer, waits until the live Application carries it,
and then runs the same undeploy.

Until the undeploy job has removed it, a file here must stay: an automated sync
of a new revision would re-create a deleted Application, and with it the
cluster. If a file is ever removed by hand, the cluster keeps running until its
Application is deleted.

The job's permission to delete Applications is the `aap-cluster-deploy-apps`
Role in `../aap-jobs/cluster-deploy-apps-role.yaml`.
