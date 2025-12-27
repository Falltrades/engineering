# [How do we convert code to production ?](./introduction.html)

## Problem

Originally Cloud Pi Native [Socle](https://github.com/cloud-pi-native/socle/) is deployed with Ansible, this involves:
- The playbook is launched after code modifications on each cluster to reconcile.
- There is no guarantee that the state of the platform on the cluster reflects that of the deployment code (what if someone manually changes something?).
- Upgrades of platform components are performed manually.

## Solution

Cloud Pi Native [Socle](https://github.com/cloud-pi-native/socle/) is based on GitOps with ArgoCD tool. Why are we not using tools we provide to our user? _Remember that the platform's engineering team must be its first customer._

Benefits to GitOps deployment:
- Automatic synchronization of all clusters after code changes on Git.
- Code = state of the platform across clusters.
- Automatic version upgrades for the various platform components (see the version range in example below).

```yaml
apiVersion: v2
dependencies:
- alias: cluster
  name: cluster
  repository: https://cloudnative-pg.io/charts
  version: '>=0.0.0 <1.0.0'
- alias: harbor
  name: harbor
  repository: https://helm.goharbor.io
  version: 1.16.0
name: harbor
type: application
version: 1.16.0
```
- Increased deployment/rollback speed.
- Improved secret management.

```yaml
cluster:
  backups:
    destinationPath: s3://<path:dso-sdid-hp/data/env/conf-dso/apps/global/values#backup | jsonPath {.s3BucketName}>/cnpg
    enabled: true
    endpointURL: <path:dso-sdid-hp/data/env/conf-dso/apps/global/values#backup | jsonPath {.s3Endpoint}>
    retentionPolicy: 14d
    s3:
      accessKey: <path:dso-sdid-hp/data/env/conf-dso/apps/global/values#backup | jsonPath {.s3AccessKey}>
      secretKey: <path:dso-sdid-hp/data/env/conf-dso/apps/global/values#backup | jsonPath {.s3SecretKey}>
```

## Issues we faced and how we solved it

| Issues | Solutions | Comment |
|:-------|:----------|:--------|
| How to manage several clusters with a common source code? | We decided to use Ansible for templating. | This was necessary to support custom configuration deployment (e.g. OpenShift, Kubernetes Vanilla). |
| How to manage secrets? | We decided to use Vault with the AVP plugin. | This allowed us to safely push our code in Github. |
| What about post-installation tasks? | We decided to run them with Kubernetes Jobs. | This reduced our workload as we are able to keep most of the code as is. |
| What about our CRD? | We decided to remove all secrets from our CRD. | Vault is now the sole source of truth for secrets. | 
| What about missing Helm Charts, or unsupported configuration? | We open a PR (if one doesn't already exist) or create our own chart | It can sometimes be difficult to get the maintainers' attention. |

Overall, the GitOps migration took longer than expected but it was definitely worth it. Now that the foundation is in place, we can tackle the [next problem](./test.html).
