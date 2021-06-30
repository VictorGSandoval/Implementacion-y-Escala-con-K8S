# Implementación y Escala con K8S

## Resumen
Kubernetes es una gran herramienta para gestionar aplicaciones e implementar una variedad de mejores prácticas de DevOps. Una de sus fortalezas es su capacidad para administrar implementaciones y escalado de aplicaciones. En esta charla, profundizaremos en cómo se ve implementar y escalar aplicaciones en un clúster de Kubernetes. Hablaremos sobre qué son las implementaciones de Kubernetes y su funcionalidad de actualización continua. También hablaremos sobre cómo puede escalar hacia arriba y hacia abajo fácilmente con las implementaciones.

## Integrantes
```
Victor Geovani SANDOVAL ROSALES
```
```
WILMER NOE Meza Yalle
```


## Referencias
- [Despliegues](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Actualizaciones continuas](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
- [Escalada](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
- [Servicio](https://kubernetes.io/docs/concepts/services-networking/service/)

## Paso a Paso
Creemos un Pod.

```
vi my-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```
kubectl apply -f my-pod.yml

kubectl get pods
```

¿Qué pasa si queremos crear múltiples réplicas del Pod?

En su lugar, intentemos una implementación.

```
vi my-deployment.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```
kubectl apply -f my-deployment.yml

kubectl get deployments

kubectl get pods
```

Ahora que tenemos varias réplicas de nuestra aplicación, necesitaremos asegurarnos de que los usuarios puedan acceder a ellas sin preocuparse por con qué Pod están interactuando. Creemos un servicio.

```
vi my-service.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-deployment
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
```

```
kubectl create -f my-service.yml

kubectl get services
```

Ahora deberíamos poder acceder a nuestra aplicación en un navegador usando el puerto `30080` en cualquiera de nuestros nodos de Kubernetes.

```
http://<node IP address>:30080
```

Ahora intentemos implementar una nueva versión.

```
vi my-deployment.yml
```

Cambie `image:` a `nginx: 1.19.10`.

```
kubectl apply -f my-deployment.yml

kubectl get pods
```

La implementación administra el proceso de implementación de la nueva versión, lo que garantiza que el pod anterior no se elimine hasta que el nuevo esté completamente listo. A esto se le llama actualización continua.

¿Qué pasa si quiero escalar y crear más pods?

```
vi my-deployment.yml
```

Cambie las `réplicas:` a `5`.

```
kubectl apply -f my-deployment.yml

kubectl get deployments

kubectl get pods
```

Puede realizar un escalado muy sencillo mediante implementaciones.

Intentemos eliminar uno de los Pods.

```
kubectl delete pod $pod_name

kubectl get pods
```

La implementación creará automáticamente un nuevo Pod para reemplazar el que se eliminó. Si quiero deshacerme de estos pods, necesitaré eliminar la implementación.

Hagamos algo un poco más complejo. ¡Hagamos una implementación azul / verde con Kubernetes!

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 2
  selector:
    matchLabels:
      split: blue
  template:
    metadata:
      labels:
        split: blue
        app: bluegreen-test
    spec:
      containers:
      - name: app
        image: linuxacademycontent/summit-k8s:blue
        ports:
        - containerPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: green
spec:
  replicas: 2
  selector:
    matchLabels:
      split: green
  template:
    metadata:
      labels:
        split: green
        app: bluegreen-test
    spec:
      containers:
      - name: app
        image: linuxacademycontent/summit-k8s:green
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: bluegreen-svc
spec:
  type: NodePort
  selector:
    app: bluegreen-test
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30081
```

Hay muchas cosas que puede hacer con las implementaciones en Kubernetes, pero esto debería darle una idea de los conceptos básicos.