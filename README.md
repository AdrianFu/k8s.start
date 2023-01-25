# k8s.start

## Open shell into a pod

```
kubectl exec --stdin --tty pod/alpine-linux-5d5f994b84-lslvx -- /bin/sh

kubectl exec learn-pod -it sh
```

## Forward local port to a pod/deployment

```
kubectl port-forward pod/.... 8080:80
kubectl port-forward deployment/.... 8080:80
```

## Create Deployment YAML with kubectl
```
kubectl create deployment nginx --image=nginx:alpine --dry-run=client -o yaml > kubectl-created.yaml
```

## YAML for service

| Content of File | Explanation |
| --------------- | ----------- |
| apiVersion: v1  | k8s API version |
| kind: Service   | resource type (Service) |
| metadata:       | Metadata about the servcie |
| spec: | |
| &nbsp;&nbsp;type: | Type of service (ClusterIP, NodePort, LoadBalancer) <br/>Default to ClusterIP |
| &nbsp;&nbsp;selector: | Select pod template label(s) that service will apply to |
| &nbsp;&nbsp;ports: | Define container target port and the port for the service |

