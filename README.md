**Proyecto 2 - ST0263: BookStore Monolítica en Clúster Kubernetes (Objetivo 3)**

**🔖 Objetivo 3 (Obligatorio)**

Desplegar la aplicación BookStore monolítica en un clúster de Kubernetes sobre AWS (EKS), conservando su estructura monolítica, sin dividir en microservicios.

-----
**📅 Contexto**

BookStore es una aplicación monolítica que simula una plataforma de venta de libros de segunda mano entre usuarios. Las funcionalidades actuales incluyen:

- Registro y autenticación de usuarios
- Visualización del catálogo de libros
- Compra, pago y envío de libros (simulado)
-----
**🛠️ Tecnologías utilizadas**

- Amazon EKS (Elastic Kubernetes Service)
- Amazon ECR (Elastic Container Registry)
- Docker
- Kubernetes (kubectl)
- AWS CLI y CloudShell
- Python Flask
- HTML/CSS
-----
**📖 Pasos realizados**

**1. Creación del entorno**

- Se accedió a AWS mediante AWS Academy.
- Se utilizó **AWS CloudShell** como entorno de trabajo por restricciones de Cloud9.

**2. Instalación de herramientas necesarias en CloudShell**

- Verificación de aws, kubectl, eksctl
- Instalación manual de eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl\_Linux\_amd64.tar.gz" -o eksctl.tar.gz

tar -xzf eksctl.tar.gz

sudo mv eksctl /usr/local/bin
```

**3. Creación del clúster Kubernetes (EKS)**

```bash
eksctl create cluster \

--name bookstore-cluster \

--region us-east-1 \

--nodes 2 \

--node-type t3.medium \

--with-oidc \

--managed
```

- El clúster se creó exitosamente con 2 nodos EC2.
- Se verificó el estado de los nodos con:

```bash
kubectl get nodes
```

**4. Preparación del código fuente**

- Se descargó y descomprimió el archivo BookStore.zip del repositorio oficial.
- Se trabajó en la carpeta BookStore-monolith.

**5. Construcción y subida de la imagen Docker**

- Se creó el repositorio en Amazon ECR:

```bash
aws ecr create-repository --repository-name bookstore --region us-east-1
```

- Se autenticó Docker en ECR:

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 644882346398.dkr.ecr.us-east-1.amazonaws.com
```

- Se construyó la imagen y se subió a ECR:
- 
```bash
docker build -t bookstore .

docker tag bookstore 644882346398.dkr.ecr.us-east-1.amazonaws.com/bookstore:latest

docker push 644882346398.dkr.ecr.us-east-1.amazonaws.com/bookstore:latest
```
**6. Despliegue de la aplicación en Kubernetes**

- Se creó el archivo deployment.yaml para definir el deployment con 2 réplicas:
  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookstore-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: bookstore
  template:
    metadata:
      labels:
        app: bookstore
    spec:
      containers:
      - name: bookstore
        image: 644882346398.dkr.ecr.us-east-1.amazonaws.com/bookstore:latest
        ports:
        - containerPort: 5000
        env:
        - name: FLASK_ENV
          value: "production"

```

- Se creó el archivo service.yaml para exponer la app vía LoadBalancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: bookstore-service
spec:
  selector:
    app: bookstore
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer

```

- Se verificó que los pods están Running y el servicio asignó una IP pública:

```bash
kubectl get pods

kubectl get svc
```

**7. Validación final**

- La aplicación se encuentra disponible en:

[BookStore](http://a6a07f04965a248f0b180017bc60d43e-1273717039.us-east-1.elb.amazonaws.com)

- Se accedió exitosamente a la interfaz principal y se confirmó el despliegue correcto.
-----

**🎓 Evidencias**

- Captura de pantalla del despliegue funcional de BookStore (interfaz web).
- 
  ![{B6E532A0-DD4A-4BBE-BBB1-A4FA25C504F3}](https://github.com/user-attachments/assets/55e79936-d963-4997-8dc1-f7bbee11290f)

- Salidas de comandos kubectl get pods, kubectl get svc con estado Running y URL activa.

```shell
BookStore-monolith $ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
bookstore-deployment-5597699d7-b5gnw   1/1     Running   0          11s
bookstore-deployment-5597699d7-c8t2v   1/1     Running   0          11s
BookStore-monolith $ kubectl get svc
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)        AGE
bookstore-service   LoadBalancer   10.100.31.215   a6a07f04965a248f0b180017bc60d43e-1273717039.us-east-1.elb.amazonaws.com   80:30671/TCP   15s
kubernetes          ClusterIP      10.100.0.1      <none>                                                                    443/TCP        21m
```
 
-----
**📆 Estado del objetivo 3:**

**Completado con éxito**: se desplegó la app monolítica BookStore en un clúster Kubernetes gestionado por EKS, utilizando buenas prácticas de contenedorización y exposición con balanceador de carga.

-----


