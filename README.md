# xtrust-gitops

Whatever is on `main` runs on the XTRUST-6G Kubernetes cluster. Argo CD applies
it within about 3 minutes of a merge.

## Before you start

Ask the cluster admin for:

- your folder name, which is also your namespace (for example `uop`)
- your read-only kubeconfig file (valid for one year), and VPN access to use it
- access to your team's private repository in XTRUST6G, used for tickets

## Deploy

1. Fork this repository (once) and create a branch. Before each new change,
   click **Sync fork** on your fork.
2. Put plain Kubernetes YAML in `apps/<your-folder>/<app-name>/`. You can copy
   [`apps/xtrust-sandbox/hello-world/`](apps/xtrust-sandbox/hello-world/hello-world.yaml)
   as a start.
3. Open a pull request against `main`. Start its title with your folder name,
   then what changes, for example `uop - add web-api`. In the description, say
   in one or two sentences what it does.
4. Wait for the admin to approve and merge it. Only the admin can merge.

To change something, edit the file in a new pull request. To remove it, delete
the file.

## Rules

- Change only your own folder.
- Leave `namespace:` out; yours is filled in automatically.
- Images must be pullable without a login.
- No passwords or tokens in plain text; see [Secrets](#secrets).
- Refused: cluster-wide objects, `Secret`, `Role`, `RoleBinding`,
  `ResourceQuota`, `LimitRange`, `NetworkPolicy`, Services of type
  `LoadBalancer` or `NodePort`.
- Pods must meet the Kubernetes
  [baseline](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline)
  level: no privileged containers, host network, host ports or host paths.
- Your pods can reach each other, DNS and the internet, but not pods in other
  namespaces. From outside they are reachable only through your `Ingress`.
- Your namespace has a quota: `kubectl describe resourcequota,limitrange`.

## Check your app

```
kubectl --kubeconfig <file> get pods
kubectl --kubeconfig <file> logs <pod-name>
kubectl --kubeconfig <file> get events
```

The kubeconfig is read-only and limited to your namespace. If nothing appears
5 minutes after the merge, [open a ticket](#problems-and-requests): the pull
request probably contains something that is refused.

## Public address

Without an `Ingress`, an app is reachable only inside the cluster. To publish it:

1. [Open a ticket](#problems-and-requests) for a hostname of the form
   `<your-folder>-<app-name>-xtrust-6g.eurodyn.com`, and say whether it is public
   or internal only. It works once the admin confirms it.
2. Add an `Ingress` with that hostname to your app, as in the
   [hello-world example](apps/xtrust-sandbox/hello-world/hello-world.yaml).
   Use no other hostname.

HTTPS is handled in front of the cluster: use plain HTTP in the Ingress and no
`tls:` section.

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

## Problems and requests

Tickets go in your team's private repository in XTRUST6G, not in this public
one. Only your team and the admins see them, and the admin is notified.

1. Open the **Issues** tab of your team's repository, for example
   `https://github.com/XTRUST6G/UOP/issues`, and click **New issue**.
2. Start the title with your folder name, then the problem, for example
   `uop - web-api does not start`.
3. Say what you expected and what happened. Add the pull request link if there
   is one, and the output of `kubectl get pods`, `kubectl get events` or the logs.
4. The admin answers in the issue and closes it when it is solved.

Open a ticket when something does not work or you need a hostname, more quota
or a new kubeconfig. Changes to your app go in a pull request, not a ticket.
Never paste passwords, tokens or your kubeconfig.
