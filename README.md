# DevOps with Kubernetes - Configurations

Codebase repository [here](https://github.com/lucksei/dwk-the-project-codebase)

Configurations for the part "4.10. The project, the grande finale" of the DevOps with Kubernetes course from Helsinki University of Finland. The original repository for the entire course's exercises can be found [here](https://github.com/lucksei/k8s-submissions-chapter2).

## Run locally

Create cluster (With k3d for example)

```sh
k3d cluster create my-cluster\
  --port 8080:80@loadbalancer \
  --port 8443:443@loadbalancer \
  --k3s-arg "--disable=traefik@server:*" \
  --agents-memory 4G \
  --servers-memory 4G \
  --agents 2
```

Install the Gateway API from kubernetes-sigs and the NGINX Gateway Fabric

```sh
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.0/standard-install.yaml
helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway --wait
```

Installing ArgoCD in the cluster [Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Run NATS messaging system

```sh
kubectl create namespace nats
helm repo add nats https://nats-io.github.io/k8s/helm/charts/
helm install my-nats nats/nats --version 2.12.2 --namespace nats
```

Deploy the applications for 'Production' and 'Staging' and the corresponding namespaces

```sh
kubectl create namespace staging
kubectl create namespace production
kubectl apply -f project/manifests/application.yaml
```

Expose the API server for ArgoCD

```sh
kubectl port-forward svc/argocd-server -n argocd 9443:443
```

Get the secret for the ArgoCD admin user

```sh
kubectl get secret argocd-initial-admin-secret --namespace argocd -o json | jq -r .data.password | base64 -d
```

Staging uses its own dedicated gateway, therefore you have to use a separate port. Test out staging using a separate dedicated port

```sh
kubectl port-forward svc/project-gateway-nginx -n staging 8081:81
```
