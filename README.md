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
- **No passwords or tokens in plain text.** This repository is public. Put them
  in as a `SealedSecret` (see [Secrets](#secrets)).
- Use container images that can be pulled without a login.
- Refused: cluster-wide objects, plain `Secret`, `Role`, `RoleBinding`,
  `ResourceQuota`, `LimitRange`, and Services of type `LoadBalancer` or `NodePort`.
- Each namespace has a quota, and containers without their own `resources` get
  small defaults. See both with `kubectl describe resourcequota,limitrange`.

## Secrets

A `SealedSecret` is an encrypted copy of a Secret. Only the cluster can decrypt
it, so it is safe in this public repository. After the merge, the cluster turns
it into a normal Secret with the same name in your namespace.

1. Install `kubeseal` once, from
   <https://github.com/bitnami-labs/sealed-secrets/releases> (`kubeseal-...` for
   your system; on macOS `brew install kubeseal`).
2. In the repository folder, encrypt the value. This runs on your computer only;
   it needs no access to the cluster.

   ```
   kubectl create secret generic my-secret --namespace <your-namespace> --from-literal=password=CHANGE-ME --dry-run=client -o yaml | kubeseal --cert sealed-secrets-cert.pem -o yaml -w apps/<your-folder>/<app-name>/my-secret.yaml
   ```

3. Commit `my-secret.yaml` and open a pull request.
4. Use the Secret in your Deployment as usual:

   ```yaml
   env:
     - name: DB_PASSWORD
       valueFrom:
         secretKeyRef:
           name: my-secret
           key: password
   ```

Good to know:

- The encrypted file works only for that secret name in that namespace.
- To change a value, run the command again and replace the file.
- To keep the value out of your shell history, put it in a file and use
  `--from-file=password=./password.txt` instead of `--from-literal`. Do not
  commit that file.
- `apps/xtrust-sandbox/hello-secret/` is a working example.

## Looking at your application

The cluster admin gives each partner a read-only kubeconfig file for its namespace:

```
kubectl --kubeconfig <file> get pods
kubectl --kubeconfig <file> logs <pod-name>
kubectl --kubeconfig <file> describe pod <pod-name>
```
