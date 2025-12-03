+++
title = "Openshift: Part 1"
date = "2025-11-18"
lastmod = 2025-11-21T12:00:00-03:00
author = "TatiMun"
keywords = ["Openshift"]
summary = "Guia para aprender lo basico de Openshift"
draft = true
type = "post"
tags = ["Kubernetes","Openshift"]
+++


Para empezar a entender Openshift, primero se debe tener los conceptos de [Contenedores](Proximamente)

Pensemos a Openshift como un restaurante enorme, donde hay mil pedidos por dia y sos una sola persona. No podes!
Necesitas:
- Cocineros
- Recetas
- Meseros
- etc.


Openshift viene a organizar tu restaurante.

Cada pod es un plato servido, un plato puede tener dos comidas (contenedores) 

Una comida tiene sus recetas, un contenedor puede tener un Nginx, otro un MySQL y asi

El pedido (Deployment) puede tener varias comidas que requieran varias recetas 
"Tengo un pedido de 3 platos de pizza listos, si uno se cae, tenemos que hacer otro para entregarlo"

Kubernetes se encarga de siempre tener los platos listos, si un plato desaparece, se crea otro.

--- 

Entonces podriamos decir que un Deployment nos da una "receta" de las aplicaciones que queremos levantar y cantidad de pods. 
Tambien podemos indicarle muchas cosas "especias"

Aca dejo un ejemplo de un deployment super optimizado:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-doble
  namespace: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-doble
  template:
    metadata:
      labels:
        app: app-doble
    spec:
      containers:
        - name: contenedor-principal
          image: nginx:1.25
          ports:
            - containerPort: 80

        - name: contenedor-secundario
          image: busybox:1.36
          command: ["sh", "-c", "while true; do echo 'Sidecar vivo'; sleep 10; done"]

```

En ese ejemplo podemos resaltar detalles importantes:

- **⭐ replicas: 1** : "Quiero tener 1 copia de este pod funcionando siempre", Kubernetes buscara la forma de tener siempre corriendo esta copia, el encargado de esto es el **ReplicaSet** (Este concepto lo veremos mas adelante)
- **⭐ contenedor-principal** : La aplicacion "real" en este caso, nginx 
- **⭐ contenedor-secundario**: Otra aplicacion real, en este caso, una app que imprime cada 10 segundos
AMBOS viven en un mismo pod (Comparten red, IP, volumenes si quisieramos)



** Debemos tener conceptos y palabras claves **
- Deployment
- Pod
- Contenedor
- Service
- Ingress
- Volume Class
- Persistent Volume 
