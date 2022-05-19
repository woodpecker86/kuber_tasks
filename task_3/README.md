# Task 3
### [Read more about CSI](https://habr.com/ru/company/flant/blog/424211/)
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
### Check pod and statefulset
```bash
kubectl get pod
kubectl get sts
```

### Homework
* We published minio "outside" using nodePort. Do the same but using ingress.
* Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.
* Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.
* Optional. Raise an nfs share on a remote machine. Create a pv using this share, create a pvc for it, create a deployment. Save data to the share, delete the deployment, delete the pv/pvc, check that the data is safe.


### That's what I've made

I've published minio "outside" via ingress and changed ingress for previous task deploy

```bash
$ curl $(minikube ip)
<!doctype html><html lang="en"><head><meta charset="utf-8"/><base href="/"/><meta content="width=device-width,initial-scale=1" name="viewport"/><meta content="#081C42" media="(prefers-color-scheme: light)" name="theme-color"/><meta content="#081C42" media="(prefers-color-scheme: dark)" name="theme-color"/><meta content="MinIO Console" name="description"/><link href="./styles/root-styles.css" rel="stylesheet"/><link href="./apple-icon-180x180.png" rel="apple-touch-icon" sizes="180x180"/><link href="./favicon-32x32.png" rel="icon" sizes="32x32" type="image/png"/><link href="./favicon-96x96.png" rel="icon" sizes="96x96" type="image/png"/><link href="./favicon-16x16.png" rel="icon" sizes="16x16" type="image/png"/><link href="./manifest.json" rel="manifest"/><link color="#3a4e54" href="./safari-pinned-tab.svg" rel="mask-icon"/><title>MinIO Console</title><script defer="defer" src="./static/js/main.069a61a0.js"></script><link href="./static/css/main.90d417ae.css" rel="stylesheet"></head><body><noscript>You need to enable JavaScript to run this app.</noscript><div id="root"><div id="preload"><img src="./images/background.svg"/> <img src="./images/background-wave-orig2.svg"/></div><div id="loader-block"><img src="./Loader.svg"/></div></div></body></html>

$ for i in {1..5}; do curl $(minikube ip)/web; done
web-6745ffd5c8-r6pfh
web-6745ffd5c8-r6pfh
web-6745ffd5c8-r6pfh
web-6745ffd5c8-nsjrr
I am canary deploy
web-6745ffd5c8-fpxmf
```
And also added envs to minio deploy and got access to the console. Result in minio-console.png

# Optional.

Created nfs server
```bash
$ kubectl apply -f nfs-server-deployment.yaml 
deployment.apps/nfs-server unchanged
$ kubectl get po -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
minio-c8dd447c-blw87         1/1     Running   0          6d    172.17.0.5    minikube   <none>           <none>
nfs-server-fb754596d-lhsv7   1/1     Running   0          15m   172.17.0.14   minikube   <none>           <none>
web-cache-6cf6588d54-ql4xk   1/1     Running   0          6d    172.17.0.12   minikube   <none>           <none>
```

Created PV
```bash
$ kubectl apply -f nfs-pv.yaml 
persistentvolume/nfs created
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                            STORAGECLASS   REASON   AGE   VOLUMEMODE
minio-deployment-pv                        5Gi        RWO            Retain           Bound       default/minio-deployment-claim                           7d    Filesystem
nfs                                        1Mi        RWX            Retain           Available                                                            5s    Filesystem
pvc-ef4224d6-1370-48d3-80d0-0a88b9ee61e5   1Gi        RWO            Delete           Bound       default/minio-minio-state-0      standard                7d    Filesystem
```

Created PVClaim
```bash
$ kubectl apply -f nfs-pvc.yaml 
persistentvolumeclaim/nfs created
$ kubectl get pvc
NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-deployment-claim   Bound    minio-deployment-pv                        5Gi        RWO                           7d1h
minio-minio-state-0      Bound    pvc-ef4224d6-1370-48d3-80d0-0a88b9ee61e5   1Gi        RWO            standard       7d
nfs                      Bound    nfs                                        1Mi        RWX                           2s
```

Created nfs test deployment - a busybox for writing into 'index.html' file.
```bash
$ kubectl apply -f nfs-test-deployment.yaml 
deployment.apps/nfs-test created
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
minio-c8dd447c-blw87         1/1     Running   0          6d
nfs-server-fb754596d-lhsv7   1/1     Running   0          16m
nfs-test-76dd5d59fc-kjxbm    1/1     Running   0          3s
nfs-test-76dd5d59fc-lkh67    1/1     Running   0          3s
web-cache-6cf6588d54-ql4xk   1/1     Running   0          6d
```
The data from nfs server
```bash
$ kubectl exec -ti nfs-server-fb754596d-lhsv7 bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
[root@nfs-server-fb754596d-lhsv7 /]# cat /exports/index.html 
Thu May 19 20:25:38 UTC 2022
Thu May 19 20:33:58 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:33:59 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:05 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:06 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:12 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:15 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:21 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:21 UTC 2022
nfs-test-76dd5d59fc-lkh67
[root@nfs-server-fb754596d-lhsv7 /]# exit
exit
```

Deleted nfs test deployment, pvc and pv
```bash
$ kubectl delete deployments.apps nfs-test 
deployment.apps "nfs-test" deleted
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
minio-c8dd447c-blw87         1/1     Running   0          6d
nfs-server-fb754596d-lhsv7   1/1     Running   0          32m
web-cache-6cf6588d54-ql4xk   1/1     Running   0          6d
$ kubectl delete pvc nfs
persistentvolumeclaim "nfs" deleted
$ kubectl get pvc
NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-deployment-claim   Bound    minio-deployment-pv                        5Gi        RWO                           7d1h
minio-minio-state-0      Bound    pvc-ef4224d6-1370-48d3-80d0-0a88b9ee61e5   1Gi        RWO            standard       7d1h
$ kubectl delete pv nfs
persistentvolume "nfs" deleted
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                            STORAGECLASS   REASON   AGE
minio-deployment-pv                        5Gi        RWO            Retain           Bound    default/minio-deployment-claim                           7d1h
pvc-ef4224d6-1370-48d3-80d0-0a88b9ee61e5   1Gi        RWO            Delete           Bound    default/minio-minio-state-0      standard                7d1h
```

The data is still there
```bash
$ kubectl exec -ti nfs-server-fb754596d-lhsv7 bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
[root@nfs-server-fb754596d-lhsv7 /]# cat /exports/index.html 
Thu May 19 20:25:38 UTC 2022
Thu May 19 20:33:58 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:33:59 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:05 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:06 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:12 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:15 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:21 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:21 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:27 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:29 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:32 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:37 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:41 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:46 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:46 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:54 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:34:55 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:34:59 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:35:03 UTC 2022
nfs-test-76dd5d59fc-kjxbm
Thu May 19 20:35:08 UTC 2022
nfs-test-76dd5d59fc-lkh67
Thu May 19 20:35:10 UTC 2022
nfs-test-76dd5d59fc-kjxb
```