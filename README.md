# ACM-Policies

A collection of **Red Hat Advanced Cluster Management (RHACM)** policy manifests for enforcing governance and resource controls across managed OpenShift clusters.

---

## Repository Structure

| File | Kind | Description |
|------|------|-------------|
| [`pvc-policy`](./pvc-policy) | `Policy` + `Placement` + `PlacementBinding` | Enforces PVC count and storage quotas on a target namespace |

---

## Policy Details

### `pvc-policy` — PVC Resource Quota Policy

Enforces storage governance on a specific namespace (`NSXXX`) across all clusters in the **global** ClusterSet. The policy creates two Kubernetes objects inside the target namespace:

| Object | Kind | Rule |
|---|---|---|
| `pvc-size-quota` | `ResourceQuota` | Max **10 PVCs** and **100Gi** total storage per namespace |
| `pvc-size-limit` | `LimitRange` | Max **12Gi** per individual PVC |

**Policy behaviour:**
- `remediationAction: enforce` — ACM will automatically create/correct the quota objects if missing or drifted
- `severity: medium`
- Targets namespace `NSXXX` — **replace with your actual namespace name before applying**
- Deployed in the `open-cluster-management-global-set` namespace
- Bound to the `global` ClusterSet via a `Placement` + `PlacementBinding`

**Apply:**
```bash
# Replace NSXXX with your target namespace first
sed -i 's/NSXXX/<your-namespace>/g' pvc-policy

oc apply -f pvc-policy
```

**Verify compliance in ACM:**
```bash
oc get policy policy-resourcequota -n open-cluster-management-global-set
```

---

## Prerequisites

- Red Hat Advanced Cluster Management (RHACM) installed on a hub cluster
- The `global` ClusterSet configured and populated with managed clusters
- `oc` CLI logged in to the hub cluster with cluster-admin privileges

---

## Customisation

To adapt `pvc-policy` for a different namespace or quota limits, update the following fields in the manifest:

| Field | Location | Description |
|---|---|---|
| `NSXXX` | `namespaceSelector`, `ResourceQuota.metadata.namespace`, `LimitRange.metadata.namespace` | Target namespace name |
| `persistentvolumeclaims: "10"` | `ResourceQuota.spec.hard` | Maximum number of PVCs allowed |
| `requests.storage: 100Gi` | `ResourceQuota.spec.hard` | Maximum total storage across all PVCs |
| `storage: 12Gi` | `LimitRange.spec.limits[0].max` | Maximum size of a single PVC |
| `clusterSets: [global]` | `Placement.spec` | Target ClusterSet(s) |

---

## License

This project is licensed under the [Apache 2.0 License](./LICENSE).
