# Node Readiness Examples (Static Pods)

This example demonstrates how NRC can be used alongside [NPD (Node Problem Detector)](https://github.com/kubernetes/node-problem-detector) to make sure that all static pods are mirrored in the API server to avoid over-committing issues. See [#115325](https://github.com/kubernetes/kubernetes/issues/115325), [#47264](https://github.com/kubernetes/website/issues/47264), and [#126870](https://github.com/kubernetes/kubernetes/pull/126870) for reference.

## Deployment Steps

1. Deploy a testing cluster. For this example we use Kind with a mounted static pod manifest for testing:

   ```bash
   kind create cluster --config examples/staticpods-readiness/kind-cluster-config.yaml
   ```

2. Install NRC:

   ```bash
   VERSION=v0.2.0
   kubectl apply -f https://github.com/kubernetes-sigs/node-readiness-controller/releases/download/${VERSION}/crds.yaml
   kubectl apply -f https://github.com/kubernetes-sigs/node-readiness-controller/releases/download/${VERSION}/install.yaml
   ```
3. Taint The node:
   ```bash
   kubectl taint node kind-worker readiness.k8s.io/StaticPodsMissing=pending:NoSchedule
   ```
4. Deploy NPD as a DaemonSet with the static pods readiness configuration:

   ```bash
   ./examples/staticpods-readiness/setup-staticpods-readiness.sh
   ```

   This deploys NPD with:
   - Init container that downloads `kubectl`, `yq`, and `jq`
   - Custom plugin that checks if static pods are mirrored
   - RBAC permissions to read pods and nodes

5. Apply the node readiness rule:
   ```bash
      kubectl apply -f examples/staticpods-readiness/staticpods-readiness-rule.yaml
   ```
   If the static pods was successfully created, NRC would read the condition added by NPD and the taint should be removed.