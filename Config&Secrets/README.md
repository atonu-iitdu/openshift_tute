
# Managing ConfigMaps and Secrets in OpenShift Deployments

This repository demonstrates how to create, inject, mount, and update ConfigMaps and Secrets in an OpenShift **Deployment**.

It covers:
- Pausing/resuming rollouts
- Creating ConfigMaps/Secrets from literals and files
- Injecting as environment variables
- Mounting as volumes
- Updating existing data
- Example YAML manifests

> Tested with `oc` CLI on OpenShift/Kubernetes-style Deployments. For DeploymentConfigs (`dc`), substitute `deploy/…` with `dc/…` where appropriate.

---

## Quick Commands

### 1) Pause rollout (safe updates)
```bash
oc rollout pause deploy/appname
```

### 2) Create ConfigMaps & Secrets

**From literals**
```bash
oc create cm mycm --from-literal=MY_STRING=foo
oc create secret generic mysecret --from-literal=MY_PASSWORD=bar
```

**From files**
```bash
oc create cm mycmfile --from-file=id_rsa.pub=~/id_rsa.pub
oc create secret generic mysecretfile --from-file=id_rsa=~/id_rsa
```

### 3) Inject as environment variables
```bash
oc set env deploy/appname --from=cm/mycm
oc set env deploy/appname --from=secret/mysecret
```

### 4) Mount as volumes (files inside the container)
```bash
oc set volume deploy/appname --add --name=mycmfile-vol   -t configmap --configmap-name=mycmfile -m /my_mount

oc set volume deploy/appname --add --name=mysecretfile-vol   -t secret --secret-name=mysecretfile -m /my_secret
```

### 5) Resume rollout (redeploy with new config)
```bash
oc rollout resume deploy/appname
```

### 6) Update existing data
```bash
oc set data cm/mycm --from-literal=MY_STRING=baz
oc set data secret/mysecret --from-literal=MY_PASSWORD=bongle

oc set data cm/mycmfile --from-file=id_rsa.pub=/etc/fstab
oc set data secret/mysecretfile --from-file=id_rsa=/etc/redhat_release
```

> **Notes**
> - Secrets are base64-encoded, not encrypted by default; ensure appropriate RBAC and, if needed, enable at-rest encryption.
> - Environment variables are convenient for simple key-value pairs. For keys/certs or structured config, prefer mounting as files.
> - If you use **DeploymentConfig** instead of **Deployment**, many commands are analogous but with `dc/NAME` instead of `deploy/NAME`.

---

## Example Manifests

See the [`manifests/`](./manifests) folder for ready-to-apply YAMLs:

- [`configmap.yaml`](./manifests/configmap.yaml) – ConfigMap with literal and file-style data
- [`secret.yaml`](./manifests/secret.yaml) – Opaque Secret with literals
- [`deployment-example.yaml`](./manifests/deployment-example.yaml) – Deployment demonstrating `envFrom` and `volumeMounts`

Apply them:
```bash
oc apply -f manifests/configmap.yaml
oc apply -f manifests/secret.yaml
oc apply -f manifests/deployment-example.yaml
```
