# How to create an argocd application in your k3s cluster

## Install Argo CD

Steps followed from https://argo-cd.readthedocs.io/en/stable/getting_started/

Create the namespace
```console
kubectl create namespace argocd
```

Apply the official installation manifests to deploy Argo CD:
```console
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

You should have argocd pods running now, validate it with:
```console
kubectl get pods -n argocd
```

##  Enable the Insecure Flag

Save the following YAML content into a file named argocd-cmd-params-cm.yaml:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cmd-params-cm
  namespace: argocd
data:
  server.insecure: "true"
```

Apply the changes to your cluster:
```console
kubectl apply -f argocd-cmd-params-cm.yaml
```

Restart the Argo CD server deployment to apply the new configuration:
```console
kubectl rollout restart deployment argocd-server -n argocd
```

Check the pod arguments to confirm the --insecure flag is active:
```console
kubectl describe pod argocd-server-55f7644948-d7bpf -n argocd
```
        
## Set Up Ingress for Access

Save the following YAML content into a file named argocd-ingress.yaml:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - host: argocd.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 80
```

Apply the Ingress manifest:
```console
kubectl apply -f ingress.yml
```
Check ingress is created:
```console
kubectl get ingress -n argocd
```

In the machine you will access argocd web server, add the following entry to your /etc/hosts file:
```console
192.168.1.100 argocd.local
```
Replace 192.168.1.100 with the IP address of your cluster.

## Clean Up
Run this command to delete all resources in the argocd namespace:
```console
kubectl delete all --all -n argocd
```

Ensure all resources are deleted:
```console
kubectl get all --all-namespaces | grep argocd
```
