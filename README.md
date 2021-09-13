kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml



kubectl create -f ./dashboard-admin-user.yml -f ./dashboard-admin-user-role.yml


kubectl -n kubernetes-dashboard describe secret admin-user-token | grep ^token


http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/


kubectl get pod -o wide

kubectl get service --all-namespaces

## Reference

https://kubernetes.io/docs/concepts/services-networking/ingress/


https://k3d.io/usage/guides/exposing_services/

## k3d exposing services ##

Create a cluster, mapping the ingress port 80 to localhost:8081

k3d cluster create --api-port 6550 -p "8081:80@loadbalancer" --agents 2


```
apiVersion: networking.k8s.io/v1beta1 # for k3s < v1.19
```
Create a cluster, mapping the ingress port 80 to localhost:8081

k3d cluster create --api-port 6550 -p "8081:80@loadbalancer" --agents 2

Good to know

--api-port 6550 is not required for the example to work.
It’s used to have k3s‘s API-Server listening on port 6550 with that port mapped to the host system.
the port-mapping construct 8081:80@loadbalancer means:
“map port 8081 from the host to port 80 on the container which matches the nodefilter loadbalancer“
the loadbalancer nodefilter matches only the serverlb that’s deployed in front of a cluster’s server nodes
all ports exposed on the serverlb will be proxied to the same ports on all server nodes in the cluster
Get the kubeconfig file (redundant, as k3d cluster create already merges it into your default kubeconfig file)

export KUBECONFIG="$(k3d kubeconfig write k3s-default)"

Create a nginx deployment

kubectl create deployment nginx --image=nginx

Create a ClusterIP service for it

kubectl create service clusterip nginx --tcp=80:80

Create an ingress object for it by copying the following manifest to a file and applying with kubectl apply -f thatfile.yaml

Note: k3s deploys traefik as the default ingress controller
```
# apiVersion: networking.k8s.io/v1beta1 # for k3s < v1.19
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```
Curl it via localhost

curl localhost:8081/

curl localhost:8081/
