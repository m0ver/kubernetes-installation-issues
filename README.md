# Solutions for saving your time on the below issues

## 1. Kubernetes stucking in Docker Desktop (MacOS)

* It might caused by a network issue
* It might happened after upgrade

Check if these docker images are pulled successfully (e.g. v1.19.3):
``` 
  k8s.gcr.io/kube-proxy               v1.19.3	cdef7632a242	10 months ago	117.69 MB	
  k8s.gcr.io/kube-apiserver           v1.19.3	a301be0cd44b	10 months ago	118.78 MB	
  k8s.gcr.io/kube-scheduler           v1.19.3	aaefbfa906bd	10 months ago	45.66 MB	
  k8s.gcr.io/kube-controller-manager  v1.19.3	9b60aca1d818	10 months ago	110.81 MB	
  k8s.gcr.io/etcd                     3.4.13-0	0369cf4303ff	11 months ago	253.39 MB
  k8s.gcr.io/pause                    3.2	80d28bedfe5d	over 1 year ago	682.7 KB
  k8s.gcr.io/coredns                  1.7.0	bfe3a36ebd25	about 1 year ago	45.23 MB
```
If you could not get those images pulled down, then use a possbile local mirror for your docker engine. You can find it by clicking `Preferences...` menu item.
Here is an example:
```
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "experimental": true,
  "debug": true
}
```
After restarting, the issue is still there. then you can try to clean up something from your local by using the below commands:
```
rm -rf ~/.kube
rm -rf ~/Library/Group\ Containers/group.com.docker/pki/
```
Then it would work for you after restarting.
## 2. Fast Installation for Kubernetes Dashboard
Kubernetes Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage applications running in the cluster and troubleshoot them, as well as manage the cluster itself.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
kubectl get pod -n kubernetes-dashboard
kubectl proxy
```
Now access Dashboard at:
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.
## 3. Create An Authentication Token (RBAC) for Kubernetes
Please follow this document: https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

## 4. Ingress pod stuck in `Pending` state due to nodeSelector: 0/1 nodes are available: 1 node(s) didn't match node selector.
You might got the issue after you deploy the ingress by the commandline:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.48.1/deploy/static/provider/cloud/deploy.yaml
```
```
kubectl label node --all kubernetes.io/os=linux 
kubectl patch deployment ingress-nginx-controller -p '{"spec":{"template":{"spec":{"nodeSelector":{"kubernetes.io/os":"linux"}}}}}

```
## 5. Failed to pull image "k8s.gcr.io/ingress-nginx/controller:v0.48.1@sha256:...": rpc error: code = Unknown desc = Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
Go to https://hub.docker.com/, and search 'ingress-nginx', choose one as below:
```
docker pull willdockerhub/ingress-nginx-controller:v0.48.1
docker tag willdockerhub/ingress-nginx-controller:v0.48.1 k8s.gcr.io/ingress-nginx/controller:v0.48.1
docker rmi willdockerhub/ingress-nginx-controller:v0.48.1
```
## 6. 0/1 nodes are available: 1 Insufficient cpu.
Resource issue, It can be fixed by reducing the cpu, or deleting the node and recreate it once again.
## 7. Ingress can not be accessed in your local with http://localhost?
If you are following the yaml from the above, you might not notice that ingress service is defined with NodePort type, which actually doesn't work. you need to change it to be LoadBalancer like below:
```
kind: Service
metadata:
  annotations:
  labels:
    helm.sh/chart: ingress-nginx-3.34.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.48.1
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller
```
Before you start to apply this configuration, you need to remove the old one first with the command:
```
kubectl delete service ingress-nginx-controller -n ingress-nginx
```
Then apply the above file.

## 8. Error from server (InternalError): error when creating "ingress.yml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1beta1/ingresses?timeout=10s": x509: certificate signed by unknown authority
It could be fixed by this command:
```
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```
## 9. <error: endpoints "*service" not found>
You need make sure the ingress is running in the same namespace as service.
