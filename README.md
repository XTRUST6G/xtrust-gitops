# xtrust-gitops

Whatever is on `main` runs on the XTRUST-6G Kubernetes cluster. Argo CD applies
it within about 3 minutes of a merge.

## Before you start

Ask the cluster admin for:

- your folder name, which is also your namespace (for example `uop`)
- your read-only kubeconfig file, and VPN access to use it

## Deploy

1. Fork this repository and create a branch.
2. Put plain Kubernetes YAML in `apps/<your-folder>/<app-name>/`. You can copy
   [`apps/xtrust-sandbox/hello-world/`](apps/xtrust-sandbox/hello-world/hello-world.yaml)
   as a start.
3. Open a pull request against `main`. It is deployed once the admin approves
   and merges it.

To change something, edit the file in a new pull request. To remove it, delete
the file.

## Rules

- Change only your own folder.
- Leave `namespace:` out; yours is filled in automatically.
- Images must be pullable without a login.
- No passwords or tokens in plain text; see [Secrets](#secrets).
- Refused: cluster-wide objects, `Secret`, `Role`, `RoleBinding`,
  `ResourceQuota`, `LimitRange`, Services of type `LoadBalancer` or `NodePort`.
- Your namespace has a quota: `kubectl describe resourcequota,limitrange`.

## Check your app

```
kubectl --kubeconfig <file> get pods
kubectl --kubeconfig <file> logs <pod-name>
kubectl --kubeconfig <file> get events
```

The kubeconfig is read-only and limited to your namespace. If nothing appears
5 minutes after the merge, ask the admin: the pull request probably contains
something that is refused.

## Secrets

Commit a secret encrypted, as a `SealedSecret`. Only the cluster can decrypt it;
there it becomes a normal Secret with the same name.

1. Install `kubeseal` from
   <https://github.com/bitnami-labs/sealed-secrets/releases>.
2. In the repository folder, run (works offline):

   ```
   kubectl create secret generic my-secret --namespace <your-folder> --from-literal=password=CHANGE-ME --dry-run=client -o yaml | kubeseal --cert sealed-secrets-cert.pem -o yaml -w apps/<your-folder>/<app-name>/my-secret.yaml
   ```

3. Add `my-secret.yaml` to your pull request and use the Secret as usual, for
   example with `secretKeyRef` (`name: my-secret`, `key: password`).

The file works only for that secret name and namespace. For a new value, run the
command again. Example: `apps/xtrust-sandbox/hello-secret/`.
