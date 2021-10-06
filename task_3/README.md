# Task 3
### Create pv in kubernetes
```bash
kubectl apply -f pv.yaml
```
### Check our pv
```bash
kubectl get pv
```
### Sample output
```bash
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
minio-deployment-pv   5Gi        RWO            Retain           Available                                   5s
```
### Create pvc
```bash
kubectl apply -f pvc.yaml
```
### Check our output in pv 
```bash
kubectl get pv
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                            STORAGECLASS   REASON   AGE
minio-deployment-pv   5Gi        RWO            Retain           Bound    default/minio-deployment-claim                           94s
```
Output is change. PV get status bound.
### Check pvc
```bash
kubectl get pvc
NAME                     STATUS   VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-deployment-claim   Bound    minio-deployment-pv   5Gi        RWO                           79s
```
### Apply deployment minio
```bash
kubectl apply -f deployment.yaml
```
### Apply svc nodeport
```bash
kubectl apply -f minio-nodeport.yaml
```
Open minikup_ip:node_port in you browser
### Apply statefulset
```bash
kubectl apply -f statefulset.yaml
```
### Ceck pod and statefulset
```bash
kubectl get pod
kubectl get sts
```

### Homework
* Мы публиковали minio "наружу" используя nodePort. Сделайте тоже самое но с использованием ingress.
* Опубликовать minio через ingress так, чтобы одновременно был доступен minio по ip_minikube и nginx, возвращающий hostname (предыдущее задание) по пути ip_minikube/web.
* Создать pv/pvc с emptyDir, создать соответствующий deployment, сохранить данные в mountPoint emptyDir, удалить поды, проверить данные.
* Опционально. Поднять nfs шару на сторонней машине. Создать pv, использующий эту шару, создать для него pvc, создать деплоймент. Сохранить данные в шару, удалить деплоймент, удалить pv/pvc, проверить сохранность данных.