# xtrust-gitops

Deployments for the XTRUST-6G Kubernetes cluster. Whatever is on the `main` branch
is what runs on the cluster: Argo CD reads this repository and applies it.

## Layout

```
apps/<namespace>/<app>/*.yaml
```

Each partner has one folder under `apps/`, named like its namespace on the cluster.

## How to deploy

1. Fork this repository and create a branch.
2. Add or change plain Kubernetes YAML files under `apps/<your-folder>/<app-name>/`.
3. Open a pull request against `main`.
4. After the pull request is approved and merged, the change is on the cluster
   within about 3 minutes.

To remove something from the cluster, delete its file in a pull request.

## Rules

- Change only your own folder.
- Leave `namespace:` out of your manifests; your namespace is filled in
  automatically. Manifests that point to another namespace are refused.
- **No passwords, tokens or Secrets.** This repository is public. Ask the cluster
  admin to create the Secret in your namespace and refer to it by name.
- Refused: cluster-wide objects, `Role`, `RoleBinding`, `ResourceQuota`,
  `LimitRange`, and Services of type `LoadBalancer` or `NodePort`.
- Each namespace has a quota, and containers without their own `resources` get
  small defaults. See both with `kubectl describe resourcequota,limitrange`.

## Looking at your application

The cluster admin gives each partner a read-only kubeconfig file for its namespace:

```
kubectl --kubeconfig <file> get pods
kubectl --kubeconfig <file> logs <pod-name>
kubectl --kubeconfig <file> describe pod <pod-name>
```
