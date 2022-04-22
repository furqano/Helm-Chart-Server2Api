# Helm-Chart-Server2Api

* Created Helm chart https://helm.sh/docs/helm/helm_create/ and modified to run server2API application.

* For Database Mongodb using community edition to create seprate chart.

## Values.yaml

```

replicaCount: 1

restapi:
  name: restapi-app
  image: furqano/restapi-app:latest
  SPRING_DATA_MONGODB_URI: mongodb://my-user:my-password@dep-mongodb.dep.svc.cluster.local:27017/server2api
  MONGODB_HOST: mongodb
  containerport: 6035
  pullPolicy: IfNotPresent

restapiservice:
  name: rest-api-app-service
  type: NodePort
  port: 6035
  protocol: TCP

DBconnect:
  UR: mongodb://
  IP: lib-mongodb.test.svc.cluster.local
  User: my-user
  Pass: my-password
  db: server2api

  
```
#### Using DNS name for Mongodb as default value to connect the application. Also parameterized the 'SPRING_DATA_MONGODB_URI' to provide custom URL.

## Chart.yaml - To add dependencies in current chart
```
#dependencies:
 # - name: mongodb
  #  repository: https://charts.bitnami.com/bitnami
   # version: ~*
```
## Template
## restapi-app.yaml (deployment.yaml)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  labels:
    {{- include "test.labels" . | nindent 4}}
spec:
  replicas: {{ .Values.replicaCount}}
  selector:
    matchLabels:
      {{- include "test.selectorLabels" . | nindent 6}}
  template:
    metadata:
      labels:
        {{- include "test.selectorLabels" . | nindent 8}}
    spec:
      containers:
      - name: {{ .Values.restapi.name}}
        image: {{ .Values.restapi.image}}
        imagePullPolicy: {{ .Values.restapi.pullPolicy}}
        env:
        - name: MONGODB_HOST
          value: {{ .Values.restapi.MONGODB_HOST}}
        - name: SPRING_DATA_MONGODB_URI
          value: {{ .Values.restapi.SPRING_DATA_MONGODB_URI}}
        - name: MONGO_USER
          value: {{ .Values.DBconnect.User}}
        - name: MONGO_PASS
          value: {{ .Values.DBconnect.Pass}}
        ports:
        - containerPort: {{ .Values.restapi.containerport}}
```

## restapiService.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.restapiservice.name}}
  labels:
    {{- include "test.labels" . | nindent 4}}
spec:
  type: {{ .Values.restapiservice.type}}
  ports:
  - port: {{ .Values.restapiservice.port}}
    protocol: {{ .Values.restapiservice.protocol}}
  selector:
    {{- include "test.selectorLabels" . | nindent 4}}
```
## Screenshots

![image](https://user-images.githubusercontent.com/64476159/164576906-7bce0a55-08c6-4985-9002-6d5bb892f599.png)
![image](https://user-images.githubusercontent.com/64476159/164576951-5dd17822-f490-415c-a3ed-a9499f8cfab6.png)
![image](https://user-images.githubusercontent.com/64476159/164579918-46e932ce-7cef-4c02-9767-6757a9570e03.png)


