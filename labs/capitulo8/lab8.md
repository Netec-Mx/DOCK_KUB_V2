# Práctica 8. Gestión de namespaces y despliegue de recursos

## Objetivos
Al finalizar la práctica, serás capaz de:
- Dominar la **gestión de namespaces en Kubernetes** aplicando **buenas prácticas** de aislamiento y límites (ResourceQuota, LimitRange, RBAC).
- Construir una **imagen Docker**, **etiquetarla y publicarla en Docker Hub** y **desplegarla** en distintos namespaces (`dev`, `prod`) con configuraciones diferenciadas.

## Duración aproximada
- 90 minutos.

## Objetivo visual



## 📝 Tabla de ayuda:
| xxx | xxxxx|

## Instrucciones


**Prerrequisitos**
  - Visual Studio Code
  - Docker Desktop en ejecución
  - Minikube y kubectl configurados
  - Terminal **Git Bash** en VS Code
  - Cuenta en **Docker Hub**
  - Conocimientos básicos de Docker, Kubernetes y YAML
introduction:
  En Kubernetes, los **namespaces** permiten **aislar recursos y políticas** dentro de un mismo clúster. Son clave para separar **entornos** (desarrollo, pruebas, producción) y aplicar **gobernanza**. cuotas de recursos, límites por contenedor, permisos (RBAC), políticas de red y credenciales para **pull** de imágenes privadas. En esta práctica crearás tres namespaces con configuraciones diferentes y desplegarás una **app Node.js** cuya imagen publicarás previamente en **Docker Hub**.
slug: lab8
lab_number: 8
final_result: >
  Una app Node.js desplegada en **dos namespaces** con **buenas prácticas**: cuotas/limites, RBAC mínimo y manejo de imágenes desde **Docker Hub**. Validaste accesos por Service y configuraste ENV por entorno, demostrando **reutilización de manifiestos** y **aislamiento** efectivo.
notes: 
  - Ajusta **ResourceQuota/LimitRange** a la capacidad real de tu clúster.
  - En producción usa **Ingress + TLS** en lugar de NodePort.
  - Versiona YAMLs en Git y usa herramientas de overlays (Kustomize/Helm) para parametrización por entorno.
  - Nunca guardes contraseñas en texto plano en repo; usa **Sealed Secrets** o gestores de secretos (Vault, SOPS).
references:
  - text: Namespaces
    url: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
  - text: ResourceQuota
    url: https://kubernetes.io/docs/concepts/policy/resource-quotas/
  - text: LimitRange
    url: https://kubernetes.io/docs/concepts/policy/limit-range/
  - text: NetworkPolicy
    url: https://kubernetes.io/docs/concepts/services-networking/
  - text: RBAC
    url: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
  - text: ImagePullSecrets
    url: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  - text: Docker Hub
    url: https://docs.docker.com/docker-hub/
prev: /capitulo7/lab7          
next: /capitulo9/lab9/



### Tarea 1. Crear la estructura del proyecto

Organizar carpetas y archivos para la app, manifestos de K8s y configuración por namespace.

**Paso 1.** Inicia sesión en tu máquina de trabajo como usuario con permisos administrativos.  

**Paso 2.** Abre el **`Visual Studio Code`** lo puedes encontrar en el **Escritorio** del ambiente o puedes buscarlo en las aplicaciones de Windows.

**Paso 3.** Una vez abierto **VSCode** da clic en el icono de la imagen para abrir la terminal, se encuentra en la parte superior derecha.

  ![micint]({{ page.images_base | relative_url }}/28.png)

**Paso 4.** Usa la terminal de **`Git Bash`**, da clic como lo muestra la imagen.

  ![micint]({{ page.images_base | relative_url }}/29.png)

**Paso 5.** Asegurate de estar dentro de la carpeta del curso llamada **dockerlabs** en la terminal de **VSCode**:

  > **NOTA:** Si te quedaste en el directorio de una práctica, usa **`cd ..`** para volver a la raíz de laboratorios.
  {: .lab-note .info .compact}

  ![micint]({{ page.images_base | relative_url }}/1.png)

**Paso 6.** Crea el directorio para trabajar en la **práctica**.

  > **NOTA:** Aislar cada práctica evita colisiones de archivos y facilita montar rutas con precisión.
  {: .lab-note .info .compact}

  ```bash
  mkdir lab8-k8snamespace && cd lab8-k8snamespace
  ```

**Paso 7.** Valida en el **Explorador** de archivos dentro de VSCode que se haya creado el directorio:

  > **NOTA:** Trabajar en VS Code permite editar y versionar cómodamente. **Git Bash** brinda compatibilidad con comandos POSIX.
  {: .lab-note .info .compact}

  ![micint]({{ page.images_base | relative_url }}/2.png)

**Paso 8.** Crearás la siguiente estructura inicial del proyecto de la aplicación:

  > **NOTA:**  
  - `base/` contiene manifests genéricos.  
  - `namespaces/` define los entornos.  
  - `policies/` y `rbac/` aplican gobernanza por entorno.
  {: .lab-note .info .compact}

  ```text
  lab8-k8snamespace/
  ├── app/
  │   ├── package.json
  │   ├── server.js
  │   └── Dockerfile
  ├── k8s/
  │   ├── base/
  │   │   ├── deployment.yaml
  │   │   └── service.yaml
  │   ├── namespaces/
  │   │   ├── dev.yaml
  │   │   └── prod.yaml
  │   ├── policies/
  │   │   ├── dev/
  │   │   │   ├── resourcequota.yaml
  │   │   │   ├── limitrange.yaml
  │   │   └── prod/
  │   │       ├── resourcequota.yaml
  │   │       ├── limitrange.yaml
  │   └── rbac/
  │       ├── dev-rbac.yaml
  │       └── prod-rbac.yaml
  └── .dockerignore

  ```
  
**Paso 9.** Ahora crea la carpeta **app/** y sus archivos vacios.

  > **NOTA:** El comando se ejecuta desde la raíz de la carpeta **lab8-k8snamespace**.
  {: .lab-note .info .compact}

  ```bash
  mkdir -p app && touch app/package.json app/server.js app/Dockerfile
  ```

**Paso 10.** Muy bien continua la creación del directorio **k8s/** con los manifiestos vacios.

  > **NOTA:** El comando se ejecuta desde la raíz de la carpeta **lab8-k8snamespace**. Estos comandos ya crean los directorios y archivos.
  {: .lab-note .info .compact}

  ```bash
  mkdir -p k8s k8s/base k8s/namespaces k8s/policies k8s/policies/dev k8s/policies/prod k8s/rbac
  touch k8s/base/deployment.yaml k8s/base/service.yaml k8s/namespaces/dev.yaml k8s/namespaces/prod.yaml
  touch k8s/policies/dev/resourcequota.yaml k8s/policies/dev/limitrange.yaml
  touch k8s/policies/prod/resourcequota.yaml k8s/policies/prod/limitrange.yaml
  touch k8s/rbac/dev-rbac.yaml k8s/rbac/prod-rbac.yaml
  ```

**Paso 11.** Crea el ultimo archivo del proyecto **.dockerignore**

  > **NOTA:** El comando se ejecuta desde la raíz de la carpeta **lab8-k8snamespace**.
  {: .lab-note .info .compact}

  ```bash
  touch .dockerignore
  ```

**Paso 12.** Agrega el siguiente contenido al archivo **.dockerignore** para construir imágenes limpias:

  > **NOTA:** Evita copiar artefactos innecesarios hacia la imagen, manteniéndola ligera.
  {: .lab-note .info .compact}

  ```gitignore
  # evita copiar node_modules del host (causa del ELF inválido)
  app/node_modules
  **/node_modules

  # basura
  npm-debug.log
  Dockerfile
  docker-compose.yml
  .git
  .gitignore
  .DS_Store
  ```

**Paso 13.** Valida la creacion de la estructura de tu proyecto, escribe el siguiente comando.

  > **NOTA:** También puedes validarlo en el explorador de archivos de VS Code.
  {: .lab-note .info .compact}

  ```bash
  ls -la
  ```

  ![micint]({{ page.images_base | relative_url }}/3.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[0] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

### Tarea 2. Implementar y contenerizar una app Node.js

Crear una app mínima con endpoint `/` y `/health`, construir imagen Docker y preparar para publicar en Docker Hub.

**Paso 14.** Abre el archivo `app/package.json` agrega las sieguientes dependencias para la aplicación:

  ```json
  {
    "name": "ns-demo",
    "version": "1.0.0",
    "main": "server.js",
    "scripts": {
      "start": "node server.js"
    },
    "dependencies": {
      "express": "^4.18.2"
    }
  }
  ```

**Paso 15.** Abre el archivo `app/server.js` y agrega la siguiente logica:

  > **NOTA:** Es una aplicación simple que expone el texto desde el namespace implementado
  {: .lab-note .info .compact}

  ```javascript
  const express = require('express');
  const app = express();
  const PORT = process.env.PORT || 3000;
  const ENV = process.env.ENV || 'local';

  app.get('/', (_req, res) => res.send(`Hola desde ${ENV} (namespaces + buenas prácticas)`));
  app.get('/health', (_req, res) => res.json({ status: 'ok', env: ENV }));

  app.listen(PORT, () => console.log(`Servidor en puerto ${PORT} (ENV=${ENV})`));
  ```

**Paso 16.** Abre el archivo `app/Dockerfile` para desplegar un multi-stage pequeño en docker:

  > **NOTA:** App simple para observar diferencias por entorno (`ENV`) y validar con probes.
  {: .lab-note .info .compact}

  ```dockerfile
  FROM node:20-alpine AS deps
  WORKDIR /app
  COPY package*.json ./
  RUN npm install --production

  FROM node:20-alpine
  WORKDIR /app
  COPY --from=deps /app/node_modules ./node_modules
  COPY . .
  ENV PORT=3000
  EXPOSE 3000
  CMD ["node", "server.js"]
  ```

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[1] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

### Tarea 3. Publicar la imagen en Docker Hub

Compilar, etiquetar y publicar la imagen para que Kubernetes pueda hacer **pull** sin depender de Minikube.

**Paso 17.** Estas son **2 opciones** para ingresa a tu cuenta de **Docker Hub** dependiendo de tu caso.

  - Si ya tienes cuenda da clic <a href="https://login.docker.com/u/login/identifier?state=hKFo2SB1QXpUSzVVc3ZDYTAzQzlkWlFoYk9LWnlLZ1VOMzNnU6Fur3VuaXZlcnNhbC1sb2dpbqN0aWTZIGx0SFpVWDNQTzNPaFZlT1FxVDlWZUpzdWUya09FaWtjo2NpZNkgbHZlOUdHbDhKdFNVcm5lUTFFVnVDMGxiakhkaTluYjk" target="_blank" rel="noopener noreferrer"><strong>AQUÍ - Iniciar Sesión</strong></a>.
  - Si no tienes cuenta da clic <a href="https://app.docker.com/signup?_gl=1*1ugpfey*_gcl_au*OTcyNTkxNjkyLjE3NTc2MDY4MTU.*_ga*MTQxMjc1NjI4My4xNzU3NjA2ODE1*_ga_XJWPQMJYHQ*czE3NTc2MTE1NTQkbzIkZzEkdDE3NTc2MTE1NTQkajYwJGwwJGgw" target="_blank" rel="noopener noreferrer"><strong>AQUÍ - Crear cuenta</strong></a>.

**Paso 18.** Ya que estes autenticado en la cuenta de **Docker Hub** da clic en **Repositories** del menu lateral izquierdo.

  ![micint]({{ page.images_base | relative_url }}/4.png)

**Paso 19.** Da clic en el botón lateral derecho **Create a repository**

  ![micint]({{ page.images_base | relative_url }}/5.png)

**Paso 20.** En el campo de **Repository Name** dale un nombre, selecciona **Public** y da clic en **Create**.

  > **NOTA:**
  - Puedes poner `kuberepo##xxx`. Cambia las `x` y `#` por letras y numeros que tu gustes.
  - Tambien puedes poner cualquier otro nombre.
  {: .lab-note .info .compact}

  ![micint]({{ page.images_base | relative_url }}/6.png)

**Paso 21.** Entra al repositorio recien creado, **copia el nombre de tu cuenta/repositorio y guardalo en un bloc de notas temporalmente**.

  ![micint]({{ page.images_base | relative_url }}/7.png)  

**Paso 22.** Ahora regresa a la terminal de **VSCode** y escribe el siguiente comando para autenticar la terminal a **Docker Hub**

  ```bash
  docker login
  ```

**Paso 23.** Sigue los pasos de la terminal para autenticarte.

  ![micint]({{ page.images_base | relative_url }}/8.png)

**Paso 24.** Si todo salio bien veras la leyenda en la terminal **Login Succeeded**

  ![micint]({{ page.images_base | relative_url }}/9.png)

**Paso 25.** Ahora si estas listo para construir la imagen Docker:

  > **IMPORTANTE:** El comando se ejecuta desde el directorio **`lab8-namespace/app`**.
  {: .lab-note .important .compact}

  ```bash
  cd app
  docker build -t ns-demo .
  ```

**Paso 26.** Lo siguiente es etiquetar la imagen para despues subirla al repositorio remoto.

  > **NOTA:**
  - Sustituye **`TU_USUARIO/TU_REPOSITORIO`** por el que guardaste en el bloc de notas.
  - El comando no dara salida a menos que haya un error.
  {: .lab-note .info .compact}

  ```bash
  docker tag ns-demo TU_USUARIO/TU_REPOSITORIO:ns-demo
  ```

  ![micint]({{ page.images_base | relative_url }}/10.png)

**Paso 27.** Si todo sale bien, el siguiente comando subira la imagen al repositorio remoto:

  > **NOTA:** Sustituye **`TU_USUARIO/TU_REPOSITORIO`** por el que guardaste en el bloc de notas.
  {: .lab-note .info .compact}

  ```bash
  docker push TU_USUARIO/TU_REPOSITORIO:ns-demo
  ```

  ![micint]({{ page.images_base | relative_url }}/11.png)

**Paso 28.** Verifica la imagen en tu cuenta <a href="https://hub.docker.com/repositories" target="_blank" rel="noopener noreferrer"><strong>Repositorios Docker Hub</strong></a>.

  > **NOTA:**
  - Publicar en **Docker Hub** facilita la portabilidad del despliegue (cualquier clúster puede usarla).
  - De igual manera puedes actualizar tu repositorio si dejaste abierta la página web.
  {: .lab-note .info .compact}

  ![micint]({{ page.images_base | relative_url }}/12.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[2] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

---

### Tarea 4. Crear namespaces y políticas base

Crear **namespaces** `dev`, `prod` y aplicar **ResourceQuota**, **LimitRange** básicos en cada uno.

#### Tarea 4.1 (Namespaces)

**Paso 29.** Abre el archivo `k8s/namespaces/dev.yaml` y agrega la siguiente configuración para crear el namespace **dev**

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: dev
    labels:
      env: dev
  ```

**Paso 30.** Abre el archivo `k8s/namespaces/prod.yaml` y agrega la siguiente configuración para crear el namespace **prod**

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: prod
    labels:
      env: prod
  ```

**Paso 31.** Ahora levanta el servicio de **Minikube**, escribe el siguiente comando

  > **NOTA:** En caso de que ya este levantado no es necesario el comando.
  {: .lab-note .info .compact}

  ```bash
  minikube start
  ```

**Paso 32.** Aplica los manifiestos para crear los **namespaces**:

  > **NOTA:** Los comandos se ejecutan desde el directorio raíz **lab8-k8snamespace**
  {: .lab-note .info .compact}

  ```bash
  cd ..
  kubectl apply -f k8s/namespaces/dev.yaml
  kubectl apply -f k8s/namespaces/prod.yaml
  ```

  ![micint]({{ page.images_base | relative_url }}/13.png)

**Paso 33.** Verifica la creación de los namespaces.

  ```bash
  kubectl get ns
  ```

  ![micint]({{ page.images_base | relative_url }}/14.png)

#### Tarea 4.2 (ResourceQuota y LimitRange NS DEV)

**Paso 34.** Abre el archivo `k8s/policies/dev/resourcequota.yaml` y agrega el siguiente control de consumo por **NS dev**:

  > **NOTA:** **ResourceQuota** controla el consumo agregado para el **namespace dev.**
  {: .lab-note .info .compact}

  ```yaml
  apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: rq-dev
    namespace: dev
  spec:
    hard:
      requests.cpu: "500m"
      requests.memory: "512Mi"
      limits.cpu: "1"
      limits.memory: "1Gi"
      pods: "10"
  ```

**Paso 35.** Abre el archivo `k8s/policies/dev/limitrange.yaml` para respaldar los limites para **NS dev**:

  > **NOTA:** **LimitRange** define requests/limits por contenedor si no se especifican.
  {: .lab-note .info .compact}

  ```yaml
  apiVersion: v1
  kind: LimitRange
  metadata:
    name: limits-dev
    namespace: dev
  spec:
    limits:
      - type: Container
        default:
          cpu: "250m"
          memory: "256Mi"
        defaultRequest:
          cpu: "100m"
          memory: "128Mi"
  ```

**Paso 36.** Aplicar los manifistos para el namespace de **dev**:

  ```bash
  kubectl apply -f k8s/policies/dev/resourcequota.yaml
  kubectl apply -f k8s/policies/dev/limitrange.yaml
  ```

  ![micint]({{ page.images_base | relative_url }}/15.png)

**Paso 37.** Verifica la creción correcta de los manifiestos en **dev**

  ```bash
  kubectl describe resourcequota rq-dev -n dev
  kubectl describe limitrange limits-dev -n dev
  ```

  ![micint]({{ page.images_base | relative_url }}/16.png)

#### Tarea 4.3 (ResourceQuota y LimitRange NS PROD)

**Paso 38.** Abre el archivo `k8s/policies/prod/resourcequota.yaml` y agrega el siguiente control de consumo por **NS prod**:

  > **NOTA:** **ResourceQuota** controla el consumo agregado para el **namespace prod.**
  {: .lab-note .info .compact}

  ```yaml
  apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: rq-prod
    namespace: prod
  spec:
    hard:
      # Totales máximos agregados del namespace (más estrictos que dev)
      requests.cpu: "2"        # dev tenía 500m → aquí 2000m para prod, pero con límites estrictos por contenedor
      requests.memory: "2Gi"   # dev 512Mi → aquí 2Gi totales
      limits.cpu: "4"
      limits.memory: "4Gi"

      # Limitar objetos (disciplina en prod)
      pods: "10"               # igual o menor que dev para forzar calidad > cantidad
      services: "10"
      services.loadbalancers: "2"
      services.nodeports: "0"  # preferible bloquear NodePorts en prod
      configmaps: "50"
      secrets: "50"
      persistentvolumeclaims: "10"
  ```

**Paso 39.** Abre el archivo `k8s/policies/prod/limitrange.yaml` para respaldar los limites para **NS prod**:

  > **NOTA:** **LimitRange** define requests/limits por contenedor si no se especifican.
  {: .lab-note .info .compact}

  ```yaml
  apiVersion: v1
  kind: LimitRange
  metadata:
    name: limits-prod
    namespace: prod
  spec:
    limits:
      - type: Container
        # Máximos permitidos por contenedor (evita “elefantes”)
        max:
          cpu: "1"         # nadie pasa de 1 vCPU por contenedor
          memory: "1Gi"    # memoria acotada por contenedor
        # Mínimos (evita contenedores “demasiado pequeños” que inestabilizan prod)
        min:
          cpu: "200m"
          memory: "256Mi"
        # Defaults si el pod no especifica (buenos valores “seguro por defecto”)
        default:
          cpu: "500m"
          memory: "512Mi"
        # Requests por defecto (lo que se reserva para planificar)
        defaultRequest:
          cpu: "300m"
          memory: "512Mi"
        # (Opcional pero recomendado) Relación máxima Limit/Request
        maxLimitRequestRatio:
          cpu: "2"         # limit ≤ 2x request
          memory: "2"
  ```

**Paso 40.** Aplicar los manifistos para el namespace de **prod**:

  ```bash
  kubectl apply -f k8s/policies/prod/resourcequota.yaml
  kubectl apply -f k8s/policies/prod/limitrange.yaml
  ```

  ![micint]({{ page.images_base | relative_url }}/17.png)

**Paso 41.** Verifica la creción correcta de los manifiestos en **prod**

  ```bash
  kubectl describe resourcequota rq-prod -n prod
  kubectl describe limitrange limits-prod -n prod
  ```

  ![micint]({{ page.images_base | relative_url }}/18.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[3] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

---

### Tarea 5. RBAC y cuentas de servicio por entorno

Crear roles y bindings mínimos para operar solo dentro del namespace correspondiente.

#### Tarea 5.1. NS DEV

**Paso 42.** Abre el archivo `k8s/rbac/dev-rbac.yaml` y agrega los siguientes permisos:

  > **NOTA:**
  - RBAC **limita privilegios** al mínimo necesario por entorno (principio de mínimo privilegio).
  - `ServiceAccount dev-sa:` Identidad que usarán los Pods/usuarios en el namespace dev
  - `Role dev-role:` Permite administrar Pods, Deployments y Services en el namespace dev
  - `RoleBinding dev-rb` Asocia el dev-role al ServiceAccount dev-sa.
  {: .lab-note .info .compact}

  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: dev-sa
    namespace: dev
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: dev-role
    namespace: dev
  rules:
    - apiGroups: ["", "apps"]
      resources: ["pods", "deployments", "services"]
      verbs: ["get", "list", "watch", "create", "update", "delete"]
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: dev-rb
    namespace: dev
  subjects:
    - kind: ServiceAccount
      name: dev-sa
      namespace: dev
  roleRef:
    kind: Role
    name: dev-role
    apiGroup: rbac.authorization.k8s.io
  ```

- **Paso 43.** Aplica el manifiesto de **RBAC** para que los permisos tomen efecto:

  ```bash
  kubectl apply -f k8s/rbac/dev-rbac.yaml
  ```

  ![micint]({{ page.images_base | relative_url }}/19.png)

**Paso 44.** Valida la creación correcta del manifiesto:

  ```bash
  kubectl get sa,role,rolebinding -n dev
  ```

  ![micint]({{ page.images_base | relative_url }}/20.png)

#### Tarea 5.2. NS PROD

**Paso 45.** Abre el archivo `k8s/rbac/prod-rbac.yaml` y agrega los siguientes permisos:

  > **NOTA:**
  - RBAC **limita privilegios** al mínimo necesario por entorno (principio de mínimo privilegio).
  - `ServiceAccount prod-sa:` Identidad que usarán los Pods/usuarios en prod
  - `Role prod-role:` Mismas APIs que en dev (pods, deployments, services).
  - `RoleBinding prod-rb` Vincula el prod-sa con permisos de solo lectura.
  {: .lab-note .info .compact}

  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: prod-sa
    namespace: prod
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: prod-role
    namespace: prod
  rules:
    - apiGroups: ["", "apps"]
      resources: ["pods", "deployments", "services"]
      verbs: ["get", "list", "watch"]   # Solo lectura en prod
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: prod-rb
    namespace: prod
  subjects:
    - kind: ServiceAccount
      name: prod-sa
      namespace: prod
  roleRef:
    kind: Role
    name: prod-role
    apiGroup: rbac.authorization.k8s.io
  ```

**Paso 46.** Aplica el manifiesto de **RBAC** para que los permisos tomen efecto:

  ```bash
  kubectl apply -f k8s/rbac/prod-rbac.yaml
  ```

  ![micint]({{ page.images_base | relative_url }}/21.png)

**Paso 47.** Valida la creación correcta del manifiesto:

  ```bash
  kubectl get sa,role,rolebinding -n prod
  ```

  ![micint]({{ page.images_base | relative_url }}/22.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[4] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}


### Tarea 6. Despliegue base (Deployment + Service) usando la imagen de Docker Hub

Definir los manifiestos base y parametrizar por namespace vía variables de entorno (ENV).

**Paso 48.** Abre el archivo `k8s/base/deployment.yaml` agrega la siguiente configuración que desplegara los pods:

  > **NOTA:**
  - Edita la **linea 18** sustituye `TU_USUARIO/TU_REPOSITORIO` por el que guardaste en el bloc de notas.
  - Es muy importante que al final mantenga `:ns-demo` si por error lo borraste agregalo
  {: .lab-note .info .compact}

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: ns-demo
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: ns-demo
    template:
      metadata:
        labels:
          app: ns-demo
      spec:
        serviceAccountName: dev-sa
        containers:
        - name: app
          image: TU_USUARIO/TU_REPOSITORIO:ns-demo
          ports:
            - containerPort: 3000
          env:
            - name: PORT
              value: "3000"
            - name: ENV
              value: "dev"
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 3
            periodSeconds: 5
  ```

**Paso 49.** Ahora en el archivo `k8s/base/service.yaml` agrega el siguiete codigo para exponer el servicio:

  > **NOTA:** Los manifiestos **base** se reutilizan para cada namespace cambiando `ENV`, `serviceAccountName` y el **namespace** de aplicación.
  {: .lab-note .info .compact}


  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: ns-demo-svc
  spec:
    type: NodePort
    selector:
      app: ns-demo
    ports:
      - port: 3000
        targetPort: 3000
        nodePort: 30081
  ```

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[5] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

---

### Tarea 7. Desplegar en cada namespace

Aplicar los manifests base en `dev` y `prod`, ajustando variables y SA.

#### Tarea 7.1. (dev)

**Paso 50.** Aplica los deployment/service en `dev`:

  ```bash
  kubectl -n dev apply -f k8s/base/deployment.yaml
  kubectl -n dev apply -f k8s/base/service.yaml
  ```

  ![micint]({{ page.images_base | relative_url }}/23.png)

**Paso 51.** Configura el ambiente para **dev**

  ```bash
  kubectl -n dev set env deployment/ns-demo ENV=dev
  kubectl -n dev patch deployment ns-demo -p '{"spec":{"template":{"spec":{"serviceAccountName":"dev-sa"}}}}'
  ```

  ![micint]({{ page.images_base | relative_url }}/24.png)

**Paso 52.** Validar la implementación correcta para ambiente **dev**:

  > **NOTA:** Aseguras que el **Service NodePort** expone la app y que `ENV=dev` se refleja en la respuesta.
  {: .lab-note .info .compact}

  ```bash
  kubectl -n dev get deploy,po,svc
  ```

  ![micint]({{ page.images_base | relative_url }}/25.png)

**Paso 53.** Expón la URL para el ambiente **dev**:

  > **NOTA:** Abre la URL que genera el comando **minikube service...** en una pestaña de tu navegador. 
  {: .lab-note .info .compact}

  ```bash
  minikube service ns-demo-svc -n dev --url
  ```

  ![micint]({{ page.images_base | relative_url }}/26.png)

#### Tarea 7.2. (prod)

**Paso 54.** Ahora en la terminal de VSCode deten el servicio corriendo para **dev** escribe `CTRL + c` y escribe el siguiente comando.

  ```bash
  kubectl -n dev delete -f k8s/base/deployment.yaml
  kubectl -n dev delete -f k8s/base/service.yaml
  ```

**Paso 55.** Aplica los deployment/service en **`prod`**:

  ```bash
  kubectl -n prod apply -f k8s/base/deployment.yaml
  kubectl -n prod apply -f k8s/base/service.yaml
  ```

  ![micint]({{ page.images_base | relative_url }}/27.png)

**Paso 56.** Notaras que el deployment se crea correcto, pero el service marca error, causa de que en el namespace **prod** esta restringido. Esto demuestra el poder de la separación de **Namespaces** con politicas.

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[6] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}

## Tarea 8. Limpieza de recursos

Siempre es importante detener y eliminar recursos creados que no se usaran.

**Paso 57.** Ejecuta el siguiente comando que eliminara todo lo que esta dentro del namespace **dev** y el namespace tambien.

  ```bash
  kubectl delete namespace dev
  ```

  ![micint]({{ page.images_base | relative_url }}/30.png)

**Paso 58.** Ejecuta el siguiente comando que eliminara todo lo que esta dentro del namespace **prod** y el namespace tambien.

  ```bash
  kubectl delete namespace prod
  ```

  ![micint]({{ page.images_base | relative_url }}/31.png)

**Paso 59.** Verifica que los **NS dev y prod**, ya no aparezcan en la lista de **namespaces**

  ```bash
  kubectl get ns
  ```

  ![micint]({{ page.images_base | relative_url }}/32.png)

{% assign results = site.data.task-results[page.slug].results %}
{% capture r1 %}{{ results[7] }}{% endcapture %}
{% include task-result.html title="Tarea finalizada" content=r1 %}
