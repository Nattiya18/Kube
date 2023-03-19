# ขั้นตอนการติดตั้ง kubectl และ minikube

* วิธีติดตั้ง kubectl บน Windows

          curl.exe -LO "https://dl.k8s.io/release/v1.26.0/bin/windows/amd64/kubectl.exe"
 
 -add path kubectl ที่ edit the system environmen
 
 -check client version

         kubectl version --client
         
* วิธีติดตั้ง Minikubes บน Windows       

# ขั้นตอนการเตรียมเครื่อง

* Windows add host ใน C:\Windows\System32\drivers\etc/hosts

# ขั้นตอน traefik

ติดตั้ง Traefik Resource Definitions

       kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v2.9/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml

ติดตั้ง RBAC 
 
       kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v2.9/docs/content/reference/dynamic-configuration/kubernetes-crd-rbac.yml
       
สร้าง namespace

       kubectl create namespace spcn16
       
ติดตั้ง Traefik Helmchart

      helm repo add traefik https://traefik.github.io/charts
      
      helm repo update
      
      helm install traefik traefik/traefik
      
เช็ค service kubectl traefik 

      kubectl get svc -l app.kubernetes.io/name=traefik
      
      kubectl get po -l app.kubernetes.io/name=traefik

หากยังไม่มีการแจก ip ให้ใช้คำสั่ง

      minikube tunnel
      
ขั้นตอนการสร้างไฟล์ secret    

     htpasswd -nB "กำหนด user" | tee auth-secret
     
     kubectl create secret generic -n "กำหนด namespace" dashboard-auth-secret --from-file=users=auth-secret -o yaml --dry-run=client | tee dashboard-secret.yaml 
     
ทำการ deploy

     kubectl apply -f traefik-dashboard.yaml
     
ทดสอบ deploy traefik ว่าสำเร็จหรือไม่โดยการใช้ domain ที่ตั้งไว้ (https://traefik.spcn16.local/dashboard/#/) 

![image](https://user-images.githubusercontent.com/119166253/226190453-a1f01cd4-e27e-4436-abc7-cd53ce3b0c6d.png)

# ขั้นตอนการ depoly hello-world-rancer

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rancher-deployment
  namespace: spcn16
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rancher
  template:
    metadata:
      labels:
        app: rancher
    spec:
      containers:
      - name: rancher
        image: rancher/hello-world
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: rancher-service
  labels:
    name: rancher-service
  namespace: spcn16
spec:
  selector:
    app: rancher
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
```

* สร้างไฟล์ rancher-hello-world.yaml
```
    apiVersion: apps/v1
kind: Deployment
metadata:
  name: rancher-deployment
  namespace: spcn16
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rancher
  template:
    metadata:
      labels:
        app: rancher
    spec:
      containers:
      - name: rancher
        image: rancher/hello-world
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: rancher-service
  labels:
    name: rancher-service
  namespace: spcn16
spec:
  selector:
    app: rancher
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
    ```
    
    

