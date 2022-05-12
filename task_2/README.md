# Task 2
### ConfigMap & Secrets
```bash
kubectl create secret generic connection-string --from-literal=DATABASE_URL=postgres://connect --dry-run=client -o yaml > secret.yaml
kubectl create configmap user --from-literal=firstname=firstname --from-literal=lastname=lastname --dry-run=client -o yaml > cm.yaml
kubectl apply -f secret.yaml
kubectl apply -f cm.yaml
kubectl apply -f pod.yaml
```
## Check env in pod
```bash
kubectl exec -it nginx -- bash
printenv
```
### Sample output (find our env)
```bash
Unable to use a TTY - input is not a terminal or the right kind of file
printenv
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
DATABASE_URL=postgres://connect
HOSTNAME=nginx
PWD=/
PKG_RELEASE=1~buster
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NJS_VERSION=0.6.2
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
lastname=lastname
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
firstname=firstname
NGINX_VERSION=1.21.3
_=/usr/bin/printenv
```
### Create deployment with simple application
```bash
kubectl apply -f nginx-configmap.yaml
kubectl apply -f deployment.yaml
```
### Get pod ip address
```bash
kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
web-5584c6c5c6-6wmdx   1/1     Running   0          4m47s   172.17.0.11   minikube   <none>           <none>
web-5584c6c5c6-l4drg   1/1     Running   0          4m47s   172.17.0.10   minikube   <none>           <none>
web-5584c6c5c6-xn466   1/1     Running   0          4m47s   172.17.0.9    minikube   <none>           <none>
```
* Try connect to pod with curl (curl pod_ip_address). What happens?
* From you PC
* From minikube (minikube ssh)
* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash)
### Create service (ClusterIP)
The command that can be used to create a manifest template
```bash
kubectl expose deployment/web --type=ClusterIP --dry-run=client -o yaml > service_template.yaml
```
Apply manifest
```bash
kubectl apply -f service_template.yaml
```
Get service CLUSTER-IP
```bash
kubectl get svc
```
### Sample output
```bash
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   20h
web          ClusterIP   10.100.170.236   <none>        80/TCP    28s
```
* Try connect to service (curl service_ip_address). What happens?

* From you PC
* From minikube (minikube ssh) (run the command several times)
* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash) (run the command several times)
### NodePort
```bash
kubectl apply -f service-nodeport.yaml
kubectl get service
```
### Sample output
```bash
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        20h
web          ClusterIP   10.100.170.236   <none>        80/TCP         15m
web-np       NodePort    10.101.147.109   <none>        80:30682/TCP   8s
```
Note how port is specified for a NodePort service
### Checking the availability of the NodePort service type
```bash
minikube ip
curl <minikube_ip>:<nodeport_port>
```
### Headless service
```bash
kubectl apply -f service-headless.yaml
```
### DNS
Connect to any pod
```bash
cat /etc/resolv.conf
```
Compare the IP address of the DNS server in the pod and the DNS service of the Kubernetes cluster.
* Compare headless and clusterip
Inside the pod run nslookup to normal clusterip and headless. Compare the results.
You will need to create pod with dnsutils.
### [Ingress](https://kubernetes.github.io/ingress-nginx/deploy/#minikube)
Enable Ingress controller
```bash
minikube addons enable ingress
```
Let's see what the ingress controller creates for us
```bash
kubectl get pods -n ingress-nginx
kubectl get pod $(kubectl get pod -n ingress-nginx|grep ingress-nginx-controller|awk '{print $1}') -n ingress-nginx -o yaml
```
Create Ingress
```bash
kubectl apply -f ingress.yaml
curl $(minikube ip)
```
### Homework
* In Minikube in namespace kube-system, there are many different pods running. Your task is to figure out who creates them, and who makes sure they are running (restores them after deletion).

* Implement Canary deployment of an application via Ingress. Traffic to canary deployment should be redirected if you add "canary:always" in the header, otherwise it should go to regular deployment.
Set to redirect a percentage of traffic to canary deployment.

### That's what I've made

The first part
```bash
$ for i in $(kubectl get po -n kube-system --output=jsonpath={.items..metadata.name}); do echo "Pod name: $i";  kubectl describe po -n kube-system $i|grep -i 'controlled by'; done
Pod name: coredns-64897985d-v92xg
Controlled By:  ReplicaSet/coredns-64897985d
Pod name: etcd-minikube
Controlled By:  Node/minikube
Pod name: kube-apiserver-minikube
Controlled By:  Node/minikube
Pod name: kube-controller-manager-minikube
Controlled By:  Node/minikube
Pod name: kube-proxy-jgnzh
Controlled By:  DaemonSet/kube-proxy
Pod name: kube-scheduler-minikube
Controlled By:  Node/minikube
Pod name: metrics-server-6cd5c97f5d-lhqpz
Controlled By:  ReplicaSet/metrics-server-6cd5c97f5d
Pod name: metrics-server-847dcc659d-8l48j
Controlled By:  ReplicaSet/metrics-server-847dcc659d
Pod name: storage-provisioner
```

The second part
Created 2 namespaces - "production" and "canary-env" and configmaps in them.
```bash
$ kubectl get configmap -A|grep nginx-configmap
canary-env             nginx-configmap                      1      17h
production             nginx-configmap                      1      17h
```

Applied deployment in namespaces
```bash
$ kubectl apply -f nginx-deployment.yaml -n production
$ kubectl apply -f nginx-deployment.yaml -n canary-env
```

Result
```bash
$ kubectl get deploy -A|grep web
canary-env             web                         2/2     2            2           18h
production             web                         2/2     2            2           18h
```

Applied services in namespaces
```bash
$ kubectl apply -f service_template.yaml -n production
$ kubectl apply -f service_template.yaml -n canary-env
```

Result
```bash
$ kubectl get service -A| grep web
canary-env             web                                  ClusterIP   10.106.230.183   <none>        80/TCP                       74m
production             web                                  ClusterIP   10.101.191.218   <none>        80/TCP                       75m
```

Applied ingresses in namespaces
```bash
$ kubectl apply -f ingress.yaml -n production
$ kubectl apply -f ingress-canary-weight.yaml -n canary-env
```

Result
```bash
$ kubectl get ingress -A
NAMESPACE    NAME                 CLASS    HOSTS   ADDRESS        PORTS   AGE
canary-env   ingress-web-weight   <none>   *       192.168.49.2   80      10m
production   ingress-web          <none>   *       192.168.49.2   80      17h
```

Work of ingress
```bash
$ for i in {1..10}; do curl -s -H "canary:never" $(minikube ip); done
web-6745ffd5c8-r6pfh
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-fpxmf
```

```bash
$ for i in {1..10}; do curl -s -H "canary:always" $(minikube ip); done
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-vpskk
I am canary deploy
web-6745ffd5c8-vpskk
I am canary deploy
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-vpskk
I am canary deploy
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-vpskk
I am canary deploy
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-vpskk
I am canary deploy
web-6745ffd5c8-vpskk
I am canary deploy
```

```bash
$ for i in {1..20}; do curl $(minikube ip); done
web-6745ffd5c8-vpskk
I am canary deploy
web-6745ffd5c8-fpxmf
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-r6pfh
web-6745ffd5c8-r6pfh
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-fpxmf
web-6745ffd5c8-fpxmf
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-r6pfh
web-6745ffd5c8-fpxmf
web-6745ffd5c8-r6pfh
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-vpskk
I am canary deploy
web-6745ffd5c8-fpxmf
```