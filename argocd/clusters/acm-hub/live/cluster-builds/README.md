# cluster-builds

One Argo CD `Application` per spoke cluster, written here by the
`gitops_cluster_build` Ansible role (`ansible/roles/gitops_cluster_build`,
template `cluster-deploy-app.yaml.j2`) when the AAP deploy job runs, and removed
by the undeploy job. The site ApplicationSet syncs without pruning, so removing a
file here does not delete its Application (or the cluster); that takes a manual
prune or delete of the `<cluster>-deploy` Application. Each renders `charts/mce-cluster-build` (or
`charts/hcp-cluster-build`) from this branch with the cluster's
`argocd/clusters/managed-clusters/<cluster>/helm-values/build-values.yaml`.

Do not add files here by hand. The directory is synced by the `acm-hub-site`
ApplicationSet (`../../site-apps.yaml`) into namespace `openshift-gitops`.
Standalone spokes are then configured by AutoShift from their entry in
`autoshift/managed-clusters.yaml`.
