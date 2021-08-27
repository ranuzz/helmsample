## k8s deployment with helm

## Create Sample App

Creating a sample flask application that runs on top of python base image.

```sh
mkdir webapp
cd webapp
python -m venv .env
.env\Scripts\activate
pip install Flask
pip freeze > requirements.txt
```

Copy the minimal application from https://flask.palletsprojects.com/en/1.1.x/quickstart/ to `app.py`

## Docker Setup

Create the dockerfile

```Dockerfile
FROM python:3.8-slim-buster

WORKDIR /app

COPY webapp/requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY webapp/app.py app.py

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=80"]
```

Test the image

```sh
docker build -t helmsample:v1 .
```

Run the container

```sh
docker run -d -p 8080:80 helmsample:v1
```

application should be running at http://localhost:8080

Stop the container

```sh
docker stop dockerId
```

## Setting up Helm

```sh
helm create deploy
```

Update few values

```diff
 image:
-  repository: nginx
+  repository: helmsample
   pullPolicy: IfNotPresent
   # Overrides the image tag whose default is the chart appVersion.
-  tag: ""
+  tag: "v1"


 ingress:
-  enabled: false
+  enabled: true
   className: ""
-  annotations: {}
-    # kubernetes.io/ingress.class: nginx
-    # kubernetes.io/tls-acme: "true"
+  annotations:
+    kubernetes.io/ingress.class: nginx
+    kubernetes.io/tls-acme: "true"
   hosts:
     - host: chart-example.local
       paths:
```

Enable Ingress Controller

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml
```

Switch to docker-desktop context

```sh
kubectl config use-context docker-desktop
```

Install helm chart

```
cd  deploy
helm install deploy ./
```

curl -v http://localhost -H "Host: chart-example.local"

Update the hosts file and you will see the page in browser as well
http://chart-example.local/