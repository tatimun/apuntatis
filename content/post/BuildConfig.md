+++
title = "BuildConfigs: La red de la construccion"
date = "2025-11-12"
lastmod = 2025-11-12
author = "TatiMun"
keywords = ["Openshift"]
summary = "¿Que es un BuildConfig?¿Se come?"
draft = false
type = "post"
tags = ["Openshift","Kubernetes"]
+++

# ✨ Build Configs ✨

Los buildConfigs son como la "receta" de lo que queremos crear, osea si yo quiero una aplicacion con nodejs pero tengo el codigo solamente por ejemplo, puedo armar una imagen con BuildConfig sin necesidad de tener que armar yo en mi pc el contenedor.

![Alt Text](https://media.tenor.com/zDlnXLw7BiEAAAAM/perfect-recipe-michael-hultquist.gif)


## Cositas importantes dentro de un BuildConfig:

- Triggers: Define cuando arrancar un build (si se cambia una imagen, si hay nuevo codigo en un repo)
- Strategy: Define COMO el build va a correr
- Sources: Define de donde obtiene la data(input)
- Output: Donde dejaria el resultado de este proceso, un contenedor listo para usar. 



Pero es necesario entender que hay diferentes formas para esta receta, depende mucho de lo que queramos obtener. 

---
**Un ejemplo**

```
kind: BuildConfig
apiVersion: build.openshift.io/v1
metadata:
  name: app1
  namespace: namespace-amb
  labels:
    app: app1
    build: app1
spec:
  nodeSelector: null
  output:
    to:
      kind: ImageStreamTag
      name: 'app1:latest'
  resources: {}
  successfulBuildsHistoryLimit: 5
  failedBuildsHistoryLimit: 5
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        namespace: openshift
        name: 'nginx:latest'
  postCommit: {}
  source:
    type: Binary
    binary: {}
  triggers:
    - type: ConfigChange
  runPolicy: Serial
```

Aca lo que vamos a tener que ver primero es: 
<p class= "highlight"> ⚠️Strategy ⚠️<style> .highlight { background-color: lightblue; font-weight: bold; text-align: center; color: black;} </style> </p>

El Strategy.Type te dice cómo va a compilar/crear la imagen. Hay diferentes tipos tambien


### ⚙️ Estrategias de Build (strategy.type)

Hay 4 tipos de Strategy:

<ol>
<li> Source (S2I)
<li>Docker
<li>Custom
<li>JenkinsPipeline
</ol>

Y cada estrategia puede combinarse con distintos source.type:
<ul>
<li> Git
<li>Binary
<li>Dockerfile
<li>Image
</ul>

<br>

---
## 1️⃣ Source Strategy (S2I)

Es el “chef clásico”:
Vos le das tu código → él lo mete en una imagen builder → cocina una imagen lista.

### Cuándo usarlo

- Cuando no querés escribir un Dockerfile.
- Cuando tu código no genera un artefacto complejo.
- Cuando querés imágenes listas para deploy al toque.

<p class="highlight"> Ejemplo con source.type: ✨Git✨ </p>

    kind: BuildConfig
    apiVersion: build.openshift.io/v1
    metadata:
    name: app-s2i-git
    spec:
    source:
        type: Git
        git:
        uri: https://github.com/tatimun/app.git
    strategy:
        type: Source
        sourceStrategy:
        from:
            kind: ImageStreamTag
            name: nodejs:16
            namespace: openshift
    output:
        to:
        kind: ImageStreamTag
        name: app-s2i-git:latest

<p class="highlight"> Ejemplo con source.type: Binary </p>

(Cuando subís el código desde tu PC/nodo o etc. con oc start-build --from-dir=.)

    kind: BuildConfig
    apiVersion: build.openshift.io/v1
    metadata:
    name: app-s2i-binary
    spec:
    source:
        type: Binary
        binary: {}
    strategy:
        type: Source
        sourceStrategy:
        from:
            kind: ImageStreamTag
            name: python:3.11
            namespace: openshift
    output:
        to:
        kind: ImageStreamTag
        name: app-s2i-binary:latest

<p class="highlight">  Ejemplo con source.type: Dockerfile </p>
(El Dockerfile está escrito dentro del BC)

    kind: BuildConfig
    apiVersion: build.openshift.io/v1
    metadata:
    name: app-s2i-inline-dockerfile
    spec:
    source:
        type: Dockerfile
        dockerfile: |
        FROM nginx:latest
        COPY . /usr/share/nginx/html
    strategy:
        type: Source
        sourceStrategy:
        from:
            kind: DockerImage
            name: nginx:latest

o con el Dockerfile en el repositorio

    kind: BuildConfig
    apiVersion: build.openshift.io/v1
    metadata:
    name: app-s2i-dockerfile-repo
    spec:
    source:
        type: Git
        git:
        uri: https://github.com/tatimun/app.git
        contextDir: "."    # donde está el Dockerfile (opcional)
    strategy:
        type: Source
        sourceStrategy:
        from:
            kind: DockerImage
            name: nginx:latest
        dockerfilePath: Dockerfile   # le indico dónde está el Dockerfile del repo
    output:
        to:
        kind: ImageStreamTag
        name: app-s2i-dockerfile-repo:latest

---
<p class="separated"> . <style> .separated { background-color: green; font-weight: bold; text-align: center; color: black;} </style> </p>

## 2️⃣ Docker Strategy

Este chef usa un Dockerfile (el tuyo o uno inline).
Todo el proceso es como un docker build, pero dentro del cluster.

### Cuándo usarlo

- Cuando ya tenés un Dockerfile.

- Cuando necesitás control total de la construcción.

- Cuando necesitás compilar artefactos complejos.

<p class="highlight">  Ejemplo con source.type: Git </p>

        kind: BuildConfig
        apiVersion: build.openshift.io/v1
        metadata:
        name: app-docker-git
        spec:
        source:
            type: Git
            git:
            uri: https://github.com/tatimun/app.git
        strategy:
            type: Docker
            dockerStrategy:
            dockerfilePath: Dockerfile
        output:
            to:
            kind: ImageStreamTag
            name: app-docker-git:latest

<p class="highlight">  Ejemplo con source.type: Binary </p>

        kind: BuildConfig
        apiVersion: build.openshift.io/v1
        metadata:
        name: app-docker-binary
        spec:
        source:
            type: Binary
        strategy:
            type: Docker

Lo triggereas con "oc start-build app-docker-binary --from-dir=. --follow"

<p class="highlight">  Ejemplo con source.type: Dockerfile inline </p>

        kind: BuildConfig
        apiVersion: build.openshift.io/v1
        metadata:
        name: app-docker-inline
        spec:
        source:
            type: Dockerfile
            dockerfile: |
            FROM registry.access.redhat.com/ubi9/nginx-122
            COPY . /usr/share/nginx/html
        strategy:
            type: Docker




























