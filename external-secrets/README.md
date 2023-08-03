# Overview

This repostiory is used to add [External Secrets](https://external-secrets.io) trait to an ACM-nabled Anthos cluster

## How to use

Must apply the following `config-management-system` namespace (no async way to do this yet):

```yaml
apiVersion: configsync.gke.io/v1beta1
kind: RootSync
metadata:
  name: es-trait-sync
  namespace: config-management-system
spec:
  sourceFormat: unstructured
  git:
    repo: "https://gitlab.com/gcp-solutions-public/retail-edge/available-cluster-traits/external-secrets-anthos.git"
    branch: "main"
    dir: "/config"
    auth: "token"
    secretRef:
      name: es-git-creds        # matches the ExternalSecret below (NOTE: Needs Google Secret Manager secret to match)

---

apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: es-git-creds-es
  namespace: config-management-system
spec:
  refreshInterval: 1m
  secretStoreRef:
    kind: ClusterSecretStore
    name: gcp-secret-store
  target:                                       # K8s secret definition
    name: es-git-creds                          ############# Matches the secretRef above
    creationPolicy: Owner
  data:
  - secretKey: username                         # K8s secret key name inside secret
    remoteRef:
      key: es-git-creds                         #  GCP Secret Name
      property: username                        # field inside GCP Secret
  - secretKey: token                            # K8s secret key name inside secret
    remoteRef:
      key: es-git-creds                         #  GCP Secret Name
      property: token                           # field inside GCP Secret

```

### Create GCP Secret

```
# One time only
gcloud secrets create es-git-creds -n config-management-system --replication-policy="automatic"
export TOKEN="<token-name>"
export TOKEN_VALUE="<token-value>"
# this can be run multiple times, adds a new version each time
echo -n "{ \"username\":\"${TOKEN}\", \"token\":\"${TOKEN_VALUE}\" }" | gcloud secrets versions add es-git-creds --data-file=-
```

## Local Validation

Assuming `nomos` is installed (via `gcloud components install nomos`)

```
nomos vet --no-api-server-check --source-format=unstructured --path config/
```

### Create/Update Config folder

```
nomos hydrate --source-format=unstructured --output=config --no-api-server-check
```

### Docker method

Using this link to find the version of nomos-docker:  https://cloud.google.com/anthos-config-management/docs/how-to/updating-private-registry#expandable-1

```
docker pull gcr.io/config-management-release/nomos:stable
docker run -it -v $(pwd):/code/ gcr.io/config-management-release/nomos:stable nomos vet --no-api-server-check --path /code/config/
```

### ACM Overview

See [our documentation](https://cloud.google.com/anthos-config-management/docs/repo) for how to use each subdirectory.
