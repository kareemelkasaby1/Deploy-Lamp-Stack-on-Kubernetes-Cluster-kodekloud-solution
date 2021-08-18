# Deploy-Lamp-Stack-on-Kubernetes-Cluster


### Secrets: 

```sh
kubectl create secret generic database --from-literal=MYSQL_ROOT_PASSWORD=123456 --from-literal=MYSQL_DATABASE=kodekloud --from-literal=MYSQL_USER=kareem --from-literal=MYSQL_PASSWORD=123456 --from-literal=MYSQL_HOST=mysql-service
```

### K8S Resources:

```sh
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: php-config
data:
  php.ini: |
     variables_order = "EGPCS"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: lamp-wp
  name: lamp-wp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lamp-wp
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: lamp-wp
    spec:
      containers:
      - image: webdevops/php-apache:alpine-3-php7
        name: httpd-php-container
        resources: {}
        ports:
        - containerPort: 80
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_USER
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_HOST
        volumeMounts:
        - name: php-ini
          mountPath: /opt/docker/etc/php/php.ini
          subPath: php.ini
      - image: mysql:5.6
        name: mysql-container
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_USER
        - name: MYSQL_HOST
          valueFrom:
            secretKeyRef:
              name: database
              key: MYSQL_HOST
        ports:
        - containerPort: 3306
        resources: {}
      volumes:
      - name: php-ini
        configMap:
          name: php-config
status: {}


---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: lamp-wp
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: lamp-service
spec:
  type: NodePort
  selector:
    app: lamp-wp
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
```

### index.php file:

```sh
<?php
$dbname = $_ENV["MYSQL_DATABASE"];
$dbuser = $_ENV["MYSQL_USER"];
$dbpass = $_ENV["MYSQL_PASSWORD"];
$dbhost = $_ENV["MYSQL_HOST"];

$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

$test_query = "SHOW TABLES FROM $dbname";
$result = mysqli_query($test_query);

if ($result->connect_error) {
   die("Connection failed: " . $conn->connect_error);
}
  echo "Connected successfully";
```
