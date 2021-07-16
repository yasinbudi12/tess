# Batumbu Test NUMBER 10

# Run Wordpress Application on Minikube

Need to create preparation yaml

- deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: {RDS Host}
        - name: WORDPRESS_DB_NAME
          value: {Name DB}
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: wordpress-db-credential
              key: DB_PASSWORD
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: wordpress-db-credential
              key: DB_USER
        volumeMounts:
        - name: wordpress-persistent
          mountPath: /var/www/html
        ports:
        - containerPort: 80
          name: wordpress
      volumes:
      - name: wordpress-persistent
        persistentVolumeClaim:
          claimName: pv-wp-claim
```
- service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
```
- secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-db-credential
type: Opaque
data:
  DB_USER: d29yZHByZXNz
  DB_PASS: Y2lsc3kxMjMh
```
- persistentvolumeclaim.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-wp-claim
  labels:
    app: wordpress
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      type: local
```
- persistentvolume.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-wordpress
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

# Minikube or Kubectl Command

Apply or Create the manifest yaml to the minikube with kubectl command.

1. Need to apply the persistent volume yaml for create persistent volume host path.
```
kubectl create -f persistentvolume.yaml
```
or
```
kubectl apply -f persistentvolume.yaml
```
2. Need to apply the persistent volume claim yaml for claim the persistent volume
```
kubectl create -f persistentvolumeclaim.yaml
```
or
```
kubectl apply -f persistentvolumeclaim.yaml
```
3. Need to apply the secret for store the sensitive or credential text.
```
kubectl create -f secret.yaml
```
or
```
kubectl apply -f secret.yaml
```
4. Need to apply the deployment yaml for create pod and replicaset service.
```
kubectl create -f deployment.yaml
```
or
```
kubectl apply -f deployment.yaml
```
5. Need to apply the service yaml for create service and the pod can be accessible from external with type Loadbalancer
```
kubectl create -f service.yaml
```
or
```
kubectl apply -f service.yaml
```

**for minikube need to run this command**
```
minikube service
```

**Scaling the pod**
```
kubectl scale deployment/wordpress --replicas=10
```

For auto scaling you need to create horizontal pod autoscaling and setup limit metric of resource pod.
