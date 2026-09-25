# Kubernetes Platform Toolkit

Small Kubernetes labs for learning how platform building blocks fit together.
Start with the Kustomize example: one application base, two environment overlays,
and manifests you can inspect without connecting to a cluster.

## Start here

Install `kubectl` with built-in Kustomize support, then run from the repository root:

```sh
kubectl kustomize examples/kustomize/overlays/dev
kubectl kustomize examples/kustomize/overlays/qa
```

These commands only render YAML. They do not deploy anything, require cluster
credentials, or install controllers. The example includes a Deployment, a
ClusterIP Service, and a Namespace.

| Environment | Namespace | Replicas |
| --- | --- | --- |
| dev | toolkit-demo-dev | 1 |
| qa | toolkit-demo-qa | 2 |

Both overlays share the same base. Change the container image in
[`base/deployment.yaml`](examples/kustomize/base/deployment.yaml), or adjust the
replica count in an overlay. The sample uses the Kubernetes documentation's
`agnhost` netexec server on port 8080, with HTTP readiness and liveness probes.
It is a learning workload, not a production platform template.

## Try it in a disposable cluster

Rendering is tested locally; cluster deployment and runtime behavior are not yet
verified in this repository. Select your own disposable cluster context before
running the following commands:

```sh
kubectl config current-context
kubectl apply -k examples/kustomize/overlays/dev
kubectl -n toolkit-demo-dev rollout status deployment/toolkit-demo --timeout=120s
kubectl -n toolkit-demo-dev port-forward service/toolkit-demo 8080:80
```

In another terminal, run `curl http://localhost:8080/hostname`. Stop the port
forward with Ctrl+C. To remove the lab:

```sh
kubectl delete -k examples/kustomize/overlays/dev
```

Cleanup also deletes the example namespace. Keep unrelated workloads out of it.

## Repository map

| Path | Purpose | Status |
| --- | --- | --- |
| [`examples/kustomize/`](examples/kustomize/) | Self-contained dev and QA lab | Start here |
| [`kustomization/`](kustomization/) | Application, ingress, autoscaling and external-secret templates | Legacy reference; contains unresolved placeholders |
| [`Ingress-controller/`](Ingress-controller/) | Historical ingress and certificate installation files | Legacy; installer changes the host and cluster |
| [`deployment-best-pratict.yaml`](deployment-best-pratict.yaml) | Deployment configuration sketch | Template; requires customization |
| [`kubernetes-cluster-with-kubeadm`](kubernetes-cluster-with-kubeadm) | Cluster bootstrap notes | Historical notes |

The legacy HPA templates use `autoscaling/v2beta2`, which Kubernetes stopped
serving in v1.26. The old installation scripts and bundled controller versions
need review before reuse; they are not part of the quickstart above.

## Contributing

Useful next steps include modernizing one legacy example at a time, adding
disposable-cluster smoke tests, and documenting controller prerequisites.
Keep changes focused and include the commands you tested in your pull request.
Do not commit cluster credentials or application secrets.

## References

- [Kubernetes: declarative management with Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
- [Kubernetes: deprecated API migration guide](https://kubernetes.io/docs/reference/using-api/deprecation-guide/)
