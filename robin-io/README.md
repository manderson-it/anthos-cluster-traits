# Overview

This is a Cluster Trait Repo suppoting the lifecycle of Robin IO SDS.


## How to use

Must apply the following `config-management-system` namespace (no async way to do this yet):


```yaml
apiVersion: configsync.gke.io/v1beta1
kind: RootSync
metadata:
  name: robin-trait-sync
  namespace: config-management-system
spec:
  git:
    repo: "https://gitlab.com/gcp-solutions-public/retail-edge/available-cluster-traits/robin-io-anthos.git"
    branch: "main"
    dir: "/config"
    auth: "token"
    secretRef:
      name: robin-git-creds # matches the external seret below

---

apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: robin-git-creds-es
  namespace: config-management-system
spec:
  refreshInterval: 1m
  secretStoreRef:
    kind: ClusterSecretStore
    name: gcp-secret-store
  target:                                       # K8s secret definition
    name: robin-git-creds                       ############# Matches the secretRef above
    creationPolicy: Owner
  data:
  - secretKey: username                         # K8s secret key name inside secret
    remoteRef:
      key: robin-access-token-creds             #  GCP Secret Name
      property: username                        # field inside GCP Secret
  - secretKey: token                            # K8s secret key name inside secret
    remoteRef:
      key: robin-access-token-creds             #  GCP Secret Name
      property: token                           # field inside GCP Secret

```

## Local Validation

Assuming `nomos` is installed (via `gcloud components install nomos`)

```
nomos vet --no-api-server-check --path config/
```

### Docker method

Using this link to find the version of nomos-docker:  https://cloud.google.com/anthos-config-management/docs/how-to/updating-private-registry#expandable-1

```
docker pull gcr.io/config-management-release/nomos:v1.13.0-rc.7
docker run -it -v $(pwd):/code/ gcr.io/config-management-release/nomos:v1.13.0-rc.7 nomos vet --no-api-server-check --path /code/config/
```

### ACM Overview

See [our documentation](https://cloud.google.com/anthos-config-management/docs/repo) for how to use each subdirectory.
