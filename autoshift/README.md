# AutoShift values

Site values for the AutoShift deployment on the `acm-hub` cluster. The policies themselves live in
the AutoShift fork (`github.com/marbindrakon/autoshiftv2`, branch `signal9`); this directory only
says which of them each cluster gets and with what settings.

| File | Scope |
|---|---|
| `global.yaml` | every cluster |
| `clustersets/hub.yaml` | the `hub` clusterset (the hub's own `local-cluster`) |
| `clustersets/managed.yaml` | the `managed` clusterset (spokes) |
| `hubs/<name>.yaml` | settings that differ per hub cluster (addresses, MACs) |
| `managed-clusters.yaml` | every spoke, one entry each (`clusters.<name>`); written by the `gitops_cluster_build` Ansible role |

The hub's `autoshift` Argo CD Application lists these files in that order; later files win.

**Nothing secret goes here.** AutoShift renders `config:` into ConfigMaps on the hub, and this
repository is public. Credentials are referenced by Secret name and come from Vault.
