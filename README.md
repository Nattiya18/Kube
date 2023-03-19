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

* สร้างไฟล์ hello-world.yaml
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
 
 
 * สร้างไฟล์ service.yaml
 ```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: service-ingress
  namespace: spcn16
spec:
  entryPoints:
    - web
    - websecure
  routes:
  - match: Host(`web.spcn16.local`)
    kind: Rule
    services:
    - name: rancher-service
      port: 80
```




# ขั้นตอนการ deploy rancher/hello-world

* deploy hello-world.yaml
 
       kubectl apply -f hello-world.yaml
    
* deploy service.yaml

       kubectl apply -f service.yaml
    
* หลังจาก deploy ให้รันคำสั่ง minikube tunnel

        minikube tunnel
    
* ทดสอบว่า deploy สำเร็จหรือไม่โดยการใช้ domain ที่ตั้งไว้ -[http://web.spcn16.local/]


# ผลลัพธ์ทั้งหมด

![image](https://user-images.githubusercontent.com/119166253/226194504-e2ed36e8-2f03-4679-a548-7f6207a6afe4.png)

![image](https://user-images.githubusercontent.com/119166253/226194534-cd80ff2b-dc8a-4bfa-8008-69b757a9a0d8.png)

![image](https://user-images.githubusercontent.com/119166253/226193916-dd0cefcf-faf3-4adf-96c8-e64961ea2216.png)
