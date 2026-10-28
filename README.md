# Set 10 — Jenkins Pipeline to configure a Kubernetes readiness probe

## File order

1. `Jenkinsfile` — checks out the repository, connects to Kubernetes, deploys the manifest, and verifies rollout/probe status.
2. `k8s/deployment.yaml` — deploys an NGINX Pod configured with an HTTP readiness probe.
3. `README.md` — setup, run, verify, and cleanup instructions.

## Before running

- Jenkins runs on Windows and the agent has `git`, `kubectl`, and access to the Kubernetes API.
- Update `GITHUB_REPO` in the Jenkinsfile to your repository URL.
- Update `KUBECONFIG_SOURCE` to the actual kubeconfig path for the Windows account whose config works with `kubectl`.
- `KUBECONFIG_DEST` is an example path for a Jenkins service account profile. If Jenkins runs under a different account, update it or use a securely configured Jenkins Secret file credential.
- The repository must contain `Jenkinsfile` and `k8s/deployment.yaml` (or `deployment.yaml` in the root).
- The Kubernetes identity needs permission to manage Deployments and Pods and read events in the target namespace.

## How the readiness probe works

The probe sends an HTTP GET request to `/` on port 80 every 5 seconds after an initial 5-second delay. Kubernetes marks the container ready when the check succeeds. If checks fail repeatedly, the Pod is removed from ready endpoints until it passes again. A readiness probe does not restart the container; liveness probes serve a different purpose.

## Run

1. Upload the files to GitHub, preserving the directory structure.
2. Edit `GITHUB_REPO`, `KUBECONFIG_SOURCE`, and if needed `KUBECONFIG_DEST` in the Jenkinsfile.
3. Create a Jenkins Pipeline job configured for the repository's `main` branch.
4. Click **Build Now**.

## Verify manually

```bat
kubectl get deployment readiness-demo
kubectl get pods -l app=readiness-demo
kubectl describe pod -l app=readiness-demo
```

Look for `Ready: True` and the readiness probe details in the Pod description.

## Cleanup

```bat
kubectl delete -f k8s/deployment.yaml
```

If your manifest is in the repository root, use `kubectl delete -f deployment.yaml`.

## Troubleshooting

- If the Pod is not ready, run `kubectl describe pod -l app=readiness-demo` and `kubectl get events --sort-by=.metadata.creationTimestamp`.
- If the image cannot be pulled, check cluster internet access and image-pull events.
- If rollout times out, inspect Pod events, container logs, and namespace permissions.
