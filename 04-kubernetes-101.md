# 04 Kubernetes 101

## Storage
### NFS Persistent Volumes

Create NFS persistent volume

```yaml
kubectl create -f - << EOF
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysqlpv-nfs
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /mnt/pool-xlab/mysqlpv
    server: 192.168.100.99
EOF

kubectl create -f - << EOF
kind: PersistentVolume
apiVersion: v1
metadata:
  name: wordpresspv-nfs
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /mnt/pool-xlab/wordpresspv
    server: 192.168.100.99
EOF

kubectl get pv
kubectl describe pv mysqlpv-nfs
kubectl delete pv mysqlpv-nfs
```
Create NFS persistent volume claim

```yaml
kubectl create -f - << EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
EOF

kubectl create -f - << EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: wordpress-pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
EOF

kubectl get pvc
kubectl describe pvc mysql-pvc-nfs
kubectl delete pvc mysql-pvc-nfs
```

Deploy MySQL
```yaml
kubectl create secret generic mysqlpwd --from-literal=password=mysql2019
kubectl get secrets

kubectl create -f - << EOF
kind: Deployment
apiVersion: apps/v1
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - image: mysql:5.6
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysqlpwd
                  key: password
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pvc-nfs
EOF

kubectl get deployments
kubectl get rs
kubectl describe deployment mysql
kubectl delete deployment mysql
kubectl get pods
kubectl describe pod mysql

kubectl create -f - << EOF
kind: Service
apiVersion: v1
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  type: ClusterIP
  ports:
    - port: 3306
  selector:
    app: mysql
EOF

showmount -e 192.168.100.99
mkdir ./nfsclient
sudo mount -t nfs -o nfsvers=4 192.168.100.99:/mnt/pool-xlab/kubepv ./nfsclient
sudo mount -t nfs4 192.168.100.99:/mnt/pool-xlab/kubepv ./nfsclient
```

Deploy WordPress
```yaml
kubectl create -f - << EOF
kind: Deployment
apiVersion: apps/v1
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
        - image: wordpress
          name: wordpress
          env:
          - name: WORDPRESS_DB_HOST
            value: mysql:3306
          - name: WORDPRESS_DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysqlpwd
                key: password
          ports:
            - containerPort: 80
              name: wordpress
          volumeMounts:
            - name: wordpress-persistent-storage
              mountPath: /var/www/html
      volumes:
        - name: wordpress-persistent-storage
          persistentVolumeClaim:
            claimName: wordpress-pvc-nfs
EOF

kubectl get pod -l app=wordpress

kubectl create -f - << EOF
kind: Service
apiVersion: v1
metadata:
  labels:
    app: wordpress
  name: wordpress
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  selector:
    app: wordpress
EOF

kubectl get svc -l app=wordpress

kubectl get pods -o=wide
kubectl delete pod -l app=mysql
kubectl delete pod -l app=wordpress

kubectl delete service wordpress
```
