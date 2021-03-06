# Helm-Chart-Server2Api

* Created Helm chart https://helm.sh/docs/helm/helm_create/ and modified to run server2API application.

* For Database Mongodb using community edition to create seprate chart.

## As Recommended to create parameterized inputs for URL in mongodb 
* Added User, Pass, IP and DB values as parameters while launching the chart. 
* Default value used is mongodb://localhost:27017/db. 
* With parameters monogdb://$(MONGO_USER):$(MONGO_PASS)@$(MONGO_IP):$(MONGO_PORT)/$(MONGO_DB)

## To Run the chart 
```
helm install my-test test --set DBconnect.IP=mongodb-ip,DBconnect.User=my-user,DBconnect.Pass=my-pass,DBconnect.db=db-name,DBconnect.Port=27017

example:

helm install my-test test --set DBconnect.IP=dep-mongodb.default.svc.cluster.local,DBconnect.User=fate,DBconnect.Pass=pass,DBconnect.db=server2api,DBconnect.Port=27017

```
## Values.yaml

```
replicaCount: 1

restapi:
  name: restapi-app
  image: furqano/test:latest
  SPRING_DATA_MONGODB_URI: ""
  MONGODB_HOST: mongodb
  containerport: 6035
  pullPolicy: Always

restapiservice:
  name: rest-api-app-service
  type: NodePort
  port: 6035
  protocol: TCP

DBconnect:
  IP: ""
  User: ""
  Pass: ""
  db: ""
  Port: 27017

```
###### Using DNS name for Mongodb as default value to connect the application. Also parameterized the 'SPRING_DATA_MONGODB_URI' to provide custom URL.
```
--set SPRING_DATA_MONGODB_URI=mongodb://my-user:my-pass@dep-mongodb.default.svc.cluster.local:27017/my-db-name
```

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
      annotations:
        rollme: {{ randAlphaNum 5 | quote }}
    spec:
      containers:
      - name: {{ .Values.restapi.name}}
        image: {{ .Values.restapi.image}}
        imagePullPolicy: {{ .Values.restapi.pullPolicy}}
        env:
        - name: MONGODB_HOST
          value: {{ .Values.restapi.MONGODB_HOST}}
        - name: MONGO_USER
          value: {{ .Values.DBconnect.User}}
        - name: MONGO_PASS
          value: {{ .Values.DBconnect.Pass}}
        - name: MONGO_DB
          value: {{ .Values.DBconnect.db}}
        - name: MONGO_PORT
          value: {{ .Values.DBconnect.Port}}
        - name: MONGO_IP
          value: {{ .Values.DBconnect.IP}}
        - name: SPRING_DATA_MONGODB_URI
          value: mongodb://$(MONGO_USER):$(MONGO_PASS)@$(MONGO_IP):$(MONGO_PORT)/$(MONGO_DB)
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

![image](https://user-images.githubusercontent.com/64476159/164786387-d9c8316c-0439-457f-8e2f-8efe7aa983c6.png)
![image](https://user-images.githubusercontent.com/64476159/164786279-620fe136-2598-4fd4-ad96-7e61ce0afb18.png)



![image](https://user-images.githubusercontent.com/64476159/164579918-46e932ce-7cef-4c02-9767-6757a9570e03.png)

## Connect to Application Using Port-Forward 

```
kubectl port-forward svc/rest-api-app-service 6035:6035
```
![port-fw ](https://user-images.githubusercontent.com/64476159/163685377-cec6506b-62cb-4be0-8477-5ce68885d42c.png)

