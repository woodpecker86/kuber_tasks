# Task 2.1
### ConfigMap & Secrets
```bash
kubectl create secret generic connection-string --from-literal=DATABASE_URL=postgres://connect --dry-run -o yaml > secret.yaml
ubectl create configmap user --from-literal=firstname=firstname --from-literal=lastname=lastname --dry-run -o yaml > cm.yaml
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
