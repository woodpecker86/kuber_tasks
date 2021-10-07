# Task 5
### Check what I can do
```bash
kubectl auth can-i create deployments --namespace kube-system
```
### Sample output
```bash
yes
```
# Настроим атентификацию пользователей с испоьзованием x509 сертификатов
### Создадим закрытый ключ
```bash
openssl genrsa -out k8s_user.key 2048
```
### Создадим запрос на подпись сертификата
```bash
openssl req -new -key k8s_user.key \
-out k8s_user.csr \
-subj "/CN=k8s_user"
```
### Подпишите CSR в Kubernetes CA. Мы должны использовать сертификат CA и ключ, которые обычно находятся в /etc/kubernetes/pki. Но т.к. мы используем minikube, то сертификаты будут лежать на хост машине в ~/.minikube
```bash
openssl x509 -req -in k8s_user.csr \
-CA ~/.minikube/ca.crt \
-CAkey ~/.minikube/ca.key \
-CAcreateserial \
-out k8s_user.crt -days 500
```
### Создадим пользователя внутри kubernetes
```bash
kubectl config set-credentials k8s_user \
--client-certificate=k8s_user.crt \
--client-key=k8s_user.key
```
### Зададим контекст для пользователя
```bash
kubectl config set-context k8s_user \
--cluster=kubernetes --user=k8s_user
```
### Далее необходимо отредактировать файл ~/.kube/config
```bash
Заменить пути на корректные
- name: k8s_user
  user:
    client-certificate: C:\Users\Andrey_Trusikhin\educ\k8s_user.crt
    client-key: C:\Users\Andrey_Trusikhin\educ\k8s_user.key
Добавить в 
contexts:
- context:
    cluster: minikube
    user: k8s_user
  name: k8s_user
```
### Переключимся на использование нового контекста
```bash
kubectl config use-context k8s_user
```
### Проверим доступы
```bash
kubectl get node
kubectl get pod
```
### Примерный вывод
```bash
Error from server (Forbidden): pods is forbidden: User "k8s_user" cannot list resource "pods" in API group "" in the namespace "default"
```
### Переключимся в админский контекст
```bash
kubectl config use-context minikube
```
### Привяжем role и clusterrole к пользователю
```bash
kubectl apply -f binding.yaml
```
### Проверим вывод команды
```bash
kubectl get pod
```
Должны увидеть доступные поды


### Homework
* Создать пользователя deploy_view и deploy_edit. Пользователю deploy_view выдать права только на просмотр деплоев, подов. Пользователю deploy_edit выдать полные права на объекты deployments, pods.
* Создать namespace prod. Создать пользователей prod_admin, prod_view. Пользователю prod_admin выдать админские права на нс prod, пользователю prod_view выдать только view права на prod.
* Создать serviceAccount sa-namespace-admin. Выдать полные права на namespace default. Создать контекст, авторизоваться используя созданный sa, проверить доступы.