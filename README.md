# sahar's assignment

## prepare minikube cluster and environment (linux)

1. install minikube
2. create a non root user
3. add the user to suoders
4. install docker
5. run `minikube start --driver docker`
6. install kubectl
7. install helm


## install nginx ingress controller

1. run `minikube addons enable ingress`
2. check if ingress pods running with `kubectl get pods -n ingress-nginx`

## create helm chart 

1. run `helm create sahar`

### prepare resources yamls 

1. create deployment yaml for both of the services

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sahar-1
  labels:
    app: svc-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: svc-1
  template:
    metadata:
      labels:
        app: svc-1
    spec:
      containers:
      - name: banana-app
        image: hashicorp/http-echo
        args:
          - "-text=banana"
```

2. create service yaml for both of the services

```
apiVersion: v1
kind: Service
metadata:
  name: sahar-svc-1
spec:
  internalTrafficPolicy: Cluster
  ports:
  - port: 5678 
    protocol: TCP
    targetPort: 5678 
  selector:
    app: svc-1
  type: ClusterIP

```

3. create ingress yaml to recive traffic

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/issuer: selfsigned-issuer
  name: sahar-ingress
spec:
  rules:
  - host: sahar-site
    http:
      paths:
      - backend:
          service:
            name: sahar-svc-1
            port:
              number: 5678
        path: /routea
        pathType: Prefix
      - backend:
          service:
            name: sahar-svc-2
            port:
              number: 5678
        path: /routeb
        pathType: Prefix
  tls:
  - hosts:
    - sahar-site
    secretName: sahar-site
```

2. convert the yamls to template with values

## deploy helm chart

1. nevigate to helm chart directory
2. run `helm install sahar-site .`

## accessing the ingress

1. run `minikube ip`
2. add the dns record in `/etc/hosts` `<minikube IP> sahar-site`
3. run `curl sahar-site/routea`
4. run `curl sahar-site/routeb`

## setup self-signed certifacte with cert-manager

1. run `kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.6.1/cert-manager.yaml`
2. create certifcate-issuer 
3. 
```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
```
4. add issuer annotation to the ingress `cert-manager.io/issuer: selfsigned-issuer`
5. run `curl -k https://sahar-site/routea`
