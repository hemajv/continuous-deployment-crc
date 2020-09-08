# Deploying a development environment

## Prequisites
* An OCP 4.x Development cluster
* Must have cluster admin (not kube:admin)

## Instructions

### Create an admin user
If you have only `kube:admin` user. You should create a new admin user.

You can use
```bash
examples/crc-create-myadmin.sh
```
to do that. It creates user named `myadmin` with password `foobar68` (change it).

Login as your new admin user.

```bash
oc login -u myadmin -p foobar68 https://api.crc.testing:6443`
```
### Create projects

Next create the project `aicoe-argocd-dev` and `argocd-test`. The latter wll be used
for deploying a dev application via ArgoCD.

```bash
oc new-project argocd-test
oc new-project aicoe-argocd-dev
```

### Deploy ArgoCD
```bash
git clone git@github.com:operate-first/continuous-deployment.git
cd continuous-deployment

# Deploy Cluser objects
kustomize build manifests/crds --enable_alpha_plugins | oc apply -f -

# Deploy Non Cluster objects
kustomize build manifests/overlays/dev --enable_alpha_plugins | oc apply -f -
```

## Configure Auth
Once deployed, there are some additional configurations, run this script:
```bash
examples/configure_development.sh
```

The script needs to be run under a user with the cluster admin role, but not with `kube:admin`.

Feel free to look inside the script for detailed comments on what configurations are applied.

## Cleanup
Run the following commands to clean up your environment.

```
kustomize build manifests/overlays/dev --enable_alpha_plugins | oc delete -f -
kustomize build manifests/crds --enable_alpha_plugins | oc delete -f -
oc delete group dev
oc delete project argocd-test
oc delete project aicoe-argocd-dev
```

You may ignore the following error when removing secrets:

```
Error from server (NotFound): error when deleting "STDIN": secrets "argocd-dex-server-oauth-token" not found
Error from server (NotFound): error when deleting "STDIN": secrets "dev-cluster-spec" not found
```
