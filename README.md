# Task 1.1
Requirements:
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
## Verify kubectl installation
```bash
kubectl version --client
```
Output, that indicates that everything is working.
```bash
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:58:59Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"windows/amd64"}
```

## Setup autocomplete for kubectl
```bash
source <(kubectl completion bash) 
```

```bash
minikube start --driver=virtualbox
```
## Get information about cluster
```bash
$ kubectl cluster-info
```
Sample output, that indicates that everything is working.
```bash
Kubernetes master is running at https://192.168.99.107:8443
CoreDNS is running at https://192.168.99.107:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'
```
## get information about available nodes
```bash
$ kubectl get nodes
```
Sample output, that indicates that everything is working.
```bash
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   9m52s   v1.22.2
```

# Install [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
```
# [More about kubernetes-dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
# Check kubernetes-dashboard ns
```bash
 kubectl get pod -n kubernetes-dashboard
```
Sample output
```bash
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-5594697f48-ng9x6   1/1     Running   0          30m
kubernetes-dashboard-57c9bfc8c8-qjt2s        1/1     Running   0          30m
```
# Install [Metrics Server](https://github.com/kubernetes-sigs/metrics-server#deployment)
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## Update deployment
```bash
kubectl edit -n kube-system deployment metrics-server
```
```bash
spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-insecure-tls
        - --kubelet-use-node-status-port
```

# Connect to Dashboard
## Get token
### Manual

```bash
kubectl describe sa -n kube-system default
# copy token name
kubectl get secrets -n kube-system
kubectl get secrets -n kube-system token_name_from_first_command -o yaml
echo -n "token_from_previous_step" | base64 -d
```
# Same thing in one command
```bash
 kubectl get secrets -n kube-system $(kubectl describe sa -n kube-system default|grep Tokens|awk '{print $2}') -o yaml|grep -E "^[[:space:]]*token:"|awk '{print $2}'|base64 -d
```

### Auto
```bash
export SECRET_NAME=$(kubectl get sa -n kube-system default -o jsonpath='{.secrets[0].name}')
export TOKEN=$(kubectl get secrets -n kube-system $SECRET_NAME -o jsonpath='{.data.token}' | base64 -d)
echo $TOKEN
```

## Connect to Dashboard
```bash
kubectl proxy
```
In browser connect to http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
# Task 1.2
# Kubernetes resourceas introduction
```bash
kubectl run web --image=nginx:latest
```
- take a look at created resource in cmd "kubectl get pods"
- take a look at created resource in Dashboard
- take a look at created resource in cmd
```bash
minikube ssh
docker container ls
```

## [Specification](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/)
```bash
kubectl explain pods.spec
```

## Create yaml manifest

Create manifest for Pod
```bash
cat > pod.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:latest
    name: nginx
EOF
```

Create manifest for ReplicaSet
```bash
cat > rs.yaml <<EOF
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: webreplica
  name: webreplica
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webreplica
  template:
    metadata:
      labels:
        app: webreplica
    spec:
      containers:
      - image: nginx:latest
        name: nginx
EOF
```

Apply manifests
```bash
kubectl apply -f pod.yaml
kubectl apply -f rs.yaml
```

```bash
kubectl run web --image=nginx:latest --dry-run=client -o yaml
```
# Task 2.1
### ConfigMap & Secrets
```bash
kubectl create secret generic connection-string --from-literal=DATABASE_URL=postgres://connect --dry-run -o yaml > secret.yaml
ubectl create configmap user --from-literal=firstname=evgenii --from-literal=lastname=mikhailov --dry-run -o yaml > cm.yaml
kubectl apply -f secret.yaml
kubectl apply -f cm.yaml
kubectl apply -f pod.yaml
```
## Проверка
```bash
kubectl exec -it nginx -- bash
printenv
```
### Создадим Deployment с простейшим приложением
```bash
kubectl apply -f nginx-configmap.yaml
kubectl apply -f deployment.yaml
```
### Получим IP адрес пода
```bash
kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
web-7db5cc6459-lp6lp   1/1     Running   0          5m50s   172.17.0.6   minikube   <none>           <none>
web-7db5cc6459-txkkq   1/1     Running   0          5m50s   172.17.0.8   minikube   <none>           <none>
web-7db5cc6459-v9r7n   1/1     Running   0          5m50s   172.17.0.7   minikube   <none>           <none>
```
Попробуйте подключиться к POD по http (curl pod_ip_address)

С вашего компьютера
Из minikube (minikube ssh)
Из другого pod (kubectl exec -it web-7db5cc6459-lp6lp bash)
### Создадим service (ClusterIP)
Команда с помощью которой можно создать заготовку манифеста
```bash
kubectl expose deployment/web --type=ClusterIP --dry-run -o yaml > service_template.yaml
```
Применим манифест
```bash
kubectl apply -f service.yaml
```
Получите CLUSTER-IP сервиса
```bash
kubectl get svc
```
Попробуйте подключиться к сервису по http (curl service_ip_address)

С вашего компьютера
Из minikube (minikube ssh)
Из другого pod (kubectl exec -it web-7db5cc6459-lp6lp bash)
### NodePort
```bash
kubectl apply -f service-nodeport.yaml
kubectl get service
```
Обратите внимание как указан port для сервиса типа NodePort
### Проверка доступности сервиса типа NodePort
```bash
minikube ip
curl <minikube_ip>:<nodeport_port>
```
### Headless сервис
```bash
kubectl apply -f service-headless.yaml
```
### DNS
Подключитесь к любому поду
```bash
cat /etc/resolv.conf
```
Сравните IP адрес DNS сервера в поде и DNS сервиса кластера Kubernetes.
### Сравнение headless и обычного clusterip
Внутри пода выполнить nslookup до обычного clusterip и headless. Сравните результат.
потребуется установить пакет dnsutils
### [Ingress](https://kubernetes.github.io/ingress-nginx/deploy/#minikube)
Включим Ingress контроллер
```bash
minikube addons enable ingress
```
Посмотрим, что создает нам ingress контроллер
```bash
kubectl get pods -n kube-system # найдем под в имени которого есть nginx-controller
kubectl get pod ingress-nginx-controller-dff6f7b7-8l7nt -n kube-system -o yaml # нужно заменить имя пода
```
Создадим Ingress
```bash
kubectl apply -f ingress.yaml
curl $(minikube ip)
```
