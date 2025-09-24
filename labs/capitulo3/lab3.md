# Práctica 3. Construcción de una imagen Docker multietapa

## 🎯 Objetivos:
Al finalizar la práctica, serás capaz de:
- Construir imágenes Docker optimizadas utilizando la técnica de **multi-stage builds**, comparando el tamaño de la imagen con y sin multietapa. Usaremos como ejemplo una aplicación Node.js con Express que sirva un frontend estático y una API REST.

## 🕒 Duración aproximada:

- 60 minutos.

## 🔍 Objetivo visual:


prerequisites:
  - Visual Studio Code instalado
  - Docker Desktop instalado y en ejecución
  - Git Bash configurado como terminal por defecto en VS Code
  - Conocimientos básicos de Node.js y Docker
  - Haber completado la practica 1
introduction:
  - Las **imágenes multi-etapa** permiten separar el proceso de construcción (instalación de dependencias y compilación de frontend) del de ejecución. Con esto se reduce el tamaño de la imagen final y se evitan dependencias innecesarias en producción. En esta práctica construiremos una aplicación Node.js con un backend REST y un frontend estático en React simulado con archivos preconstruidos, para demostrar el impacto de los multi-stage builds.
slug: lab3
lab_number: 3
final_result: >
  El estudiante habrá construido una aplicación Node.js contenerizada con un **Dockerfile multi-etapa**, obteniendo una imagen ligera y optimizada, validando que la aplicación funciona correctamente y que el tamaño de la imagen se reduce respecto a una construcción tradicional.
notes: 
  - Multi-stage es aplicable a múltiples lenguajes (Java, Go, .NET).
  - Siempre validar la funcionalidad de la aplicación antes de optimizar.
  - Para builds grandes, se recomienda usar `--target` para depuración de etapas intermedias.
references:
  - text: Docke Multi-stage builds
    url: https://docs.docker.com/develop/develop-images/multistage-build/
  - text: Docker Hub - Node Images
    url: https://hub.docker.com/_/node

## Instrucciones


### Tarea 1. Preparar la estructura del proyecto

Crear la estructura de carpetas con backend y frontend.

**Paso 1.** Inicia sesión en tu máquina de trabajo como usuario con permisos administrativos.  

**Paso 2.** Abre el **`Visual Studio Code`** lo puedes encontrar en el **Escritorio** del ambiente o puedes buscarlo en las aplicaciones de Windows.

**Paso 3.** Una vez abierto **VSCode** da clic en el icono de la imagen para abrir la terminal, se encuentra en la parte superior derecha.

  ![micint]({{ page.images_base | relative_url }}/9.png)

**Paso 4.** Usa la terminal de **`Git Bash`**, da clic como lo muestra la imagen.

  ![micint]({{ page.images_base | relative_url }}/10.png)

**Paso 5.** Asegurate de estar dentro de la carpeta del curso llamada **dockerlabs** en la terminal de **VSCode**:

  > **NOTA:** Si te quedaste en el directorio de una practica usa **`cd ..`** para retornar a la raíz de laboratorios.
  {: .lab-note .info .compact}

  ![micint]({{ page.images_base | relative_url }}/1.png)

**Paso 6.** Ahora crea el directorio para trabajar en la practica 3:

  ```bash
  mkdir lab3-dockermultistage && cd lab3-dockermultistage
  ```

**Paso 7.** Valida en el **Explorador** de archivos dentro de VSCode que se haya creado el directorio:

  ![micint]({{ page.images_base | relative_url }}/2.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[0] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}



### Tarea 2. Creación del backend

Implementar una API REST en Node.js. Se reutilizara el backend creado en la practica 1

**Paso 8.** Copia la carpeta **backend** de la practica 1, escribe el siguiente comando.

  > **IMPORTANTE:**
    - El comando se ejecuta desde adentro de la carpeta **lab3-dockermultistage**.
    - En caso de que no estes dentro de la carpeta **lab3...** ajusta las rutas.
  {: .lab-note .important .compact}


  ```bash
  cp -r ../lab1-acontactos/backend/ .
  ```

**Paso 9.** Ahora escribe el siguiente comando para validar que se haya copiado correctamente la carpeta **backend**

  > **NOTA:** Recuerda que tambien puedes visualizarlo en el explorador de archivos de VSCode.
  {: .lab-note .info .compact}

  ```bash
  ls -la
  ```

  ![micint]({{ page.images_base | relative_url }}/3.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[1] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}


### Tarea 3: Creación del frontend

Se reutilizará el directorio **frontend** que tiene el sitio web estático.

**Paso 10.** Copia la carpeta **frontend** de la practica 1, escribe el siguiente comando.

  > **IMPORTANTE:**
    - El comando se ejecuta desde adentro de la carpeta **lab3-dockermultistage**.
    - En caso de que no estes dentro de la carpeta **lab3...** ajusta las rutas.
  {: .lab-note .important .compact}

  ```bash
  cp -r ../lab1-acontactos/frontend/ .
  ```

**Paso 11.** Ahora escribe el siguiente comando para validar que se haya copiado correctamente la carpeta **frontend**

  > **NOTA:** Recuerda que tambien puedes visualizarlo en el explorador de archivos de VSCode.
  {: .lab-note .info .compact}

  ```bash
  ls
  ```

  ![micint]({{ page.images_base | relative_url }}/4.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[2] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

### Tarea 4: Construir imagen sin multi-stage

En esta tarea crearas un Dockerfile simple y analizaras el tamaño.

**Paso 12.** Crea el archivo **Dockerfile** dentro del directorio **lab3-dockermultistage.**

  > **NOTA:** El comando se ejecuta desde la carpeta **lab3...**
  {: .lab-note .info .compact}

  ```bash
  touch Dockerfile
  code Dockerfile
  ```

  ![micint]({{ page.images_base | relative_url }}/5.png)

**Paso 13.** Agrea el siguiente codigo al archivo **Dockerfile**.

  > **NOTA:** Este Dockerfile crea una imagen sin multi-stage.
  {: .lab-note .info .compact}

  ```dockerfile
  FROM node:20-alpine
  WORKDIR /app
  COPY backend/package*.json ./
  RUN npm install --production
  COPY backend ./backend
  COPY frontend ./frontend
  EXPOSE 3000
  CMD ["node", "backend/server.js"]
  ```

**Paso 14.** Compila el proyecto de docker, escribe el siguiente comando dentro de la terminal.

  > **NOTA:** El comando se ejecuta desde la carpeta **lab3...**
  {: .lab-note .info .compact}

  ```bash
  docker build -t contactos-tradicional .
  ```

**Paso 15.** Ahora valida el tamaño de la imagen creada despues de la compilación.

  > **NOTA:** Aproximadamente el tamaño quedara `207MB` puede ser diferente.
  {: .lab-note .info .compact}

  ```bash
  docker images contactos-tradicional
  ```

  ![micint]({{ page.images_base | relative_url }}/6.png)

**Paso 16.** Si es necesario anota el numero del tamaño de la imagen.

  > **NOTA:** Esta es la forma tradicional, pero genera imágenes más grandes dependiendo de como este estructurada la aplicación.
  {: .lab-note .info .compact}

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[3] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}


### Tarea 5. Construir imagen con multi-stage

En esta tarea optimizaras la construcción usando **multi-stage**.

**Paso 17.** Crea otro archivo **Dockerfile** para que quede mejor organizado, escribe el siguiente comando.

  > **NOTA:**
    - El nombre **opt** hace referencia a **Optimización** para la compilación del Multi-Stage
    - Se crea dentro del mismo directorio **lab3...** 
  {: .lab-note .info .compact}

  ```bash
  touch Dockerfile.opt
  code Dockerfile.opt
  ```

**Paso 18.** Ahora agrega el siguiente codigo a ese nuevo archivo **Dockerfile.opt**

  > **NOTA:** El contenido es muy parecido al ejemplo anterior, pero internamente se mejora el proceso reduciendo las capas.
  {: .lab-note .info .compact}

  ```dockerfile
  # Etapa 1: build
  FROM node:20-alpine AS builder
  WORKDIR /app
  COPY backend/package*.json ./
  RUN npm install --production
  COPY backend ./backend
  COPY frontend ./frontend

  # Etapa 2: ejecución
  FROM node:20-alpine
  WORKDIR /app
  COPY --from=builder /app ./ 
  EXPOSE 3000
  CMD ["node", "backend/server.js"]
  ```

**Paso 19.** Ahora construye la imagen, escribe el siguiente comando.

  > **NOTA:** Quizas alcances a notar que compila un poco mas rapido, todo depende de que tanta información tenga tu proyecto.
  {: .lab-note .info .compact}

  ```bash
  docker build -f Dockerfile.opt -t contactos-opt .
  ```

**Paso 20.** Escribe el siguiente comando para validar el tamaño de la imagen.

  ```bash
  docker images contactos-opt
  ```

  ![micint]({{ page.images_base | relative_url }}/7.png)

**Paso 21.** Si es necesario anota el numero del tamaño de la imagen.

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[4] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}


### Tarea 6. Comparar resultados

En esta tarea verificaras la diferencia de tamaños de las imagenes.

**Paso 22.** Usa el siguiente comando para realizar la comparacion de las imagnes.

  > **IMPORTANTE:**
    - El comando se ejecuta dentro de la terminal **GitBash**
    - Puede ser en cualquier directorio, pero deseado en **lab3...**
  {: .lab-note .important .compact}

  ```bash
  docker images | grep contactos-
  ```

  ![micint]({{ page.images_base | relative_url }}/8.png)

**Paso 23.** Puedes observar que ligeramente la imagen **opt** es mas pequeña, recuerda que depende de que tanta informacion tenga tu proyecto.

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[5] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}


### Tarea 7. Añadir .dockerignore (mejor caché y tamaño)

En esta tarea crearás un archivo .dockerignore para excluir archivos innecesarios y mejorar el tamaño y la caché de las imágenes.

**Paso 24.** Crea y abre un archivo **.dockerignore** en la raíz **lab3-dockermultistage**:

  ```bash
  touch .dockerignore
  code .dockerignore
  ```

**Paso 25.** Ahora agrega el siguiente contenido al archivo

  ```gitignore
  backend/node_modules
  frontend/node_modules
  /node_modules
  .git
  .vscode
  .DS_Store
  npm-debug.log
  Dockerfile*
  Thumbs.db
  ```

**Paso 26.** Reconstruye la imagen **tradicional**:

  ```bash
  docker build -t contactos-tradicional .
  ```

**Paso 27.** Reconstruye la imagen **optimizada**:

  ```bash
  docker build -f Dockerfile.opt -t contactos-opt .
  ```

**Paso 28.** Ahora valida ambos tamaños y compara el resultado:

  > **NOTA:** Como puedes observar se hizo una reducción relativa a la cantidad de archivos de la aplicación
  {: .lab-note .info .compact}

  ```bash
  docker images | grep contactos-
  ```

  ![micint]({{ page.images_base | relative_url }}/11.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[6] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

---

### Tarea 8. Variante de runtime aún más ligera

En esta tarea crearás un Dockerfile.slim con multi-stage minimalista para generar imágenes más ligeras y seguras.

**Paso 29.** Crea **Dockerfile.slim** para copiar únicamente lo necesario:

  > **NOTA:** El comando se ejecuta dentro del directorio **lab3...**
  {: .lab-note .info .compact}

  ```bash
  touch Dockerfile.slim
  ```

**Paso 30.** Agrega el siguiente contenido a ese nuevo Dockerfile.

  ```dockerfile
  FROM node:20-alpine AS deps
  WORKDIR /app
  COPY backend/package*.json ./
  RUN npm install --omit=dev
  COPY backend ./backend

  FROM node:20-alpine
  WORKDIR /app
  COPY --from=deps /app/backend ./backend
  COPY --from=deps /app/node_modules ./node_modules
  RUN addgroup -S appgrp && adduser -S appusr -G appgrp
  USER appusr
  EXPOSE 3000
  CMD ["node", "backend/server.js"]
  ```

**Paso 31.** Construye la nueva imagen basada en el **Dockerfile.slim**:

  ```bash
  docker build -f Dockerfile.slim -t contactos-slim .
  ```

**Paso 32.** Escribe el siguiente comando para observar el resultado.

  > **NOTA:** Tanto la version **opt** como la **slim** usan **Multi-Stage**. Pero **slim** es mucho mas minimalista ya que la propiedad **--from=deps** copia solo lo que se necesita.
  {: .lab-note .info .compact}

  ```bash
  docker images | grep contactos-
  ```

  ![micint]({{ page.images_base | relative_url }}/12.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[7] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

---

### Tarea 9. Limpieza y buenas prácticas

En esta tarea eliminaras las imagenes creadas para mantener limpio el ambiente.

**Paso 33.** Escribe el siguiente comando para eliminar las imagenes:

  ```bash
  docker rmi contactos-tradicional contactos-opt contactos-slim
  ```

  ![micint]({{ page.images_base | relative_url }}/13.png)

**Paso 34.** Verifica que ya no aparezca ninguna imagen **contactos-...**

  > **NOTA:** En caso de que todavia exista alguna, repite el paso anterior.
  {: .lab-note .info .compact}

  ```bash
  docker images
  ```
  ![micint]({{ page.images_base | relative_url }}/14.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[8] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}
