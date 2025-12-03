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

Para empezar con Openshift (Que es una version de Kubernetes pero version paga y mas outOfTheBox) debieramos empezar a entender, para que nos sirve y que viene a resolver. 


## 🧨 Problemática

Antes las aplicaciones corrían en servidores físicos o VMs.

Si una se caía → había que adivinar **en cuál servidor estaba**, entrar por SSH, revisar logs, ver CPU/RAM, la red, reiniciar, cruzar los dedos 😵‍💫😵

Y si necesitabas más capacidad → había que **comprar otro servidor**, configurarlo a mano y esperar que no explote todo en un deploy un viernes 18 hs.

¡Te volvias loco!


## ✨ Solución

Gracias a Kubernetes / OpenShift 🚀  
todas las aplicaciones corren en **contenedores** (consumiendo solo lo justo y necesario).

Si un pod se muere → **se levanta otro solo** 💅✨

No hay que entrar a 20 servidores distintos a ver cuál se rompió.  
Ahora **el control es centralizado**: declarás **el estado deseado** y Kubernetes se encarga de que **se cumpla siempre** 🧠

Las apps dejan de ser **mascotas** y pasan a ser **ganado**:  
si una cae, se crea otra nueva automáticamente 🐄💥

--- 

## ¡Tu propio restaurante!

Ahora, pensemos que este orquestador de contenedores en un Resto 👨‍🍳 Hay mil pedidos por dia y no llegar a cumplir con todo! Por suerte, tu ayudante te da una super ayuda.

## 🍽️ Metáfora del restaurante

Para entender Kubernetes sin llorar, pensemos en un restaurante (tu app) 😌✨💪

## 🍽️ Metáfora del restaurante — Versión mejorada 😌✨💪

Para entender Kubernetes, pensemos en un restaurante (tu aplicación) 😌✨💪

---

### 🥘 Deployment = El **plato del menú** 😌✨💪
Representa **qué** querés servir y **cuántos platos** tienen que estar listos.
Ej: *“Quiero 3 milanesas con papas listas siempre”*  
→ mantener replicas del plato

---

### 🍱 Pod = El **plato servido en la mesa** 😌✨💪
Cada Pod es **una instancia real del plato**.  
Si un plato se cae al suelo, sale otro **automáticamente** 😌✨💪

---

### 🧂 Contenedores = **Guarniciones / ingredientes del plato** 😌✨💪
Un plato puede tener:
- milanesa (app principal)
- papas fritas (sidecar: logs, proxy, etc.)

Están **en el mismo plato**, llegan **juntos** a la mesa 😌✨💪

---

### 🧑‍🍽️ Service = El **mozo** 😌✨💪
No importa **dónde** están los platos,  
el mozo **sabe a qué cocina pedirlos** y **cómo entregarlos** al cliente 😌✨💪

---

### 🚪 Ingress = La **persona de la entrada / reservas** 😌✨💪
Controla **quién entra** al restaurante  
y a **qué plato** (aplicación) quiere acceder 😌✨💪

---

### 🔥 Node = La **cocina / estación de trabajo** 😌✨💪
Donde realmente se preparan los platos.  
Si una cocina falla → movemos la preparación a otra 😌✨💪

---

### 🧠 Control Plane = El **gerente del restaurante** 😌✨💪
Supervisa todo:
- que haya suficientes platos
- que los mozos funcionen bien
- que la cocina no explote

Asegura que **el restaurante funcione aunque haya caos** 😌✨💪

---

📌 *Resumen express:*  
**Deployment** pide 5 platos → **Pods** son esos platos → **Contenedores** son la comida en el plato → **Service** los sirve → **Ingress** deja entrar a los clientes 😌✨💪


