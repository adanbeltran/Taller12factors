# Taller: Contenerización y despliegue de EcoRed con Docker Hub y Render

**Proyecto:** EcoRed Circular  
**Entorno local:** Windows 11 + Docker Desktop  
**Container Registry:** Docker Hub  
**Plataforma de despliegue:** Render  
**Arquitectura del ejercicio:** una imagen, un contenedor y un único punto de entrada HTTP  
**Nivel:** Ingeniería de Software / Cloud Computing  

---

# Tabla de contenido

1. [Introducción: de una aplicación web local a un servicio desplegado en la nube](#introducción-de-una-aplicación-web-local-a-un-servicio-desplegado-en-la-nube)
2. [Objetivo y arquitectura del taller](#objetivo-y-arquitectura-del-taller)
3. [Fase 1. Conceptos y preparación](#fase-1-conceptos-y-preparación)
4. [Fase 2. Construir y probar el contenedor localmente](#fase-2-construir-y-probar-el-contenedor-localmente)
5. [Fase 3. Publicar y validar la imagen desde Docker Hub](#fase-3-publicar-y-validar-la-imagen-desde-docker-hub)
6. [Fase 4. Desplegar la imagen en Render](#fase-4-desplegar-la-imagen-en-render)
7. [Fase 5. Pruebas finales y análisis Twelve-Factor](#fase-5-pruebas-finales-y-análisis-twelve-factor)
8. [Entregables](#entregables)
9. [Referencias técnicas](#referencias-técnicas)

---

# Introducción: de una aplicación web local a un servicio desplegado en la nube

Una aplicación web moderna no está formada únicamente por el código del frontend y del backend. Para que un usuario pueda escribir una dirección en el navegador, enviar una petición y recibir una respuesta, intervienen diferentes componentes de infraestructura y distintos tipos de servidores.

En este taller se utilizará deliberadamente una arquitectura sencilla:

```text
Navegador
   │
   │ HTTPS / HTTP
   ▼
Render
   │
   │ HTTP hacia el contenedor
   ▼
Nginx :10000
   │
   ├────────────► React compilado
   │
   └── /api/*
          │ HTTP interno
          ▼
      Gunicorn :8000
          │ WSGI
          ▼
        Django
          │
     ┌────┴────┐
     ▼         ▼
MongoDB      Firebase
 Atlas
```

Esta arquitectura permite distinguir varias responsabilidades que frecuentemente se confunden.

## Servidor HTTP o servidor web

Un **servidor HTTP** es un proceso capaz de recibir solicitudes utilizando el protocolo HTTP y devolver respuestas HTTP.

Ejemplos conocidos son:

- Nginx;
- Apache HTTP Server;
- Caddy.

Un servidor HTTP puede realizar tareas como:

- recibir conexiones;
- atender rutas;
- servir HTML, CSS, JavaScript, imágenes y otros archivos estáticos;
- aplicar encabezados HTTP;
- comprimir respuestas;
- actuar como reverse proxy;
- distribuir solicitudes entre otros procesos;
- manejar o participar en la terminación TLS/HTTPS.

En este taller **Nginx** desempeña esta función.

Para una solicitud al frontend:

```text
GET /
```

Nginx entrega directamente:

```text
index.html
JavaScript
CSS
assets
```

Para una solicitud como:

```text
GET /api/health/
```

Nginx no la procesa como lógica de negocio. La reenvía al backend.

Esta función recibe el nombre de **reverse proxy** o proxy inverso.

---

## Servidor WSGI

**WSGI** significa **Web Server Gateway Interface**.

Académicamente es importante precisar que WSGI **no es un servidor ni un protocolo de red**. Es una **especificación de interfaz para Python**, definida en PEP 3333, que establece cómo un servidor de aplicaciones debe comunicarse con una aplicación web Python.

Por ello, cuando se habla informalmente de un **“servidor WSGI”**, se hace referencia a un servidor de aplicaciones que implementa esa especificación.

Ejemplos:

- Gunicorn;
- uWSGI;
- mod_wsgi para Apache.

En este taller se utiliza:

```text
Gunicorn
```

Gunicorn recibe la petición HTTP enviada por Nginx y ejecuta la aplicación Django mediante el objeto WSGI:

```text
config.wsgi:application
```

Conceptualmente:

```text
Nginx
  │
  │ HTTP
  ▼
Gunicorn
  │
  │ llamada WSGI
  ▼
Django
```

Gunicorn también puede escuchar HTTP directamente, pero su responsabilidad principal en esta arquitectura es actuar como **servidor de aplicación WSGI** entre el servidor web frontal y Django.

---

## Diferencia entre servidor HTTP y servidor WSGI

| Aspecto | Servidor HTTP / Web | Servidor de aplicación WSGI |
|---|---|---|
| Ejemplo del taller | Nginx | Gunicorn |
| Interfaz principal | HTTP | HTTP hacia afuera y WSGI hacia Django |
| Objetivo principal | Recibir y enrutar tráfico web, servir contenido estático | Ejecutar aplicaciones Python compatibles con WSGI |
| Sirve React compilado | Sí | No es su responsabilidad principal |
| Ejecuta código Django | No | Sí |
| Reverse proxy | Sí | No es su función principal |
| Conoce la especificación WSGI | No es necesario | Sí |
| Puerto en este taller | `10000` | `8000` interno |

Por tanto, no deben interpretarse como tecnologías competidoras.

En esta arquitectura son **complementarias**:

```text
Nginx
servidor HTTP
    │
    ▼
Gunicorn
servidor de aplicación WSGI
    │
    ▼
Django
framework
```

---

## ¿Qué es ASGI y por qué no se utiliza aquí?

Django también soporta **ASGI — Asynchronous Server Gateway Interface**.

ASGI surgió para soportar mejor escenarios asincrónicos y conexiones de larga duración, por ejemplo:

- WebSockets;
- streaming;
- muchas operaciones concurrentes de I/O;
- aplicaciones con código `async/await`.

Servidores ASGI comunes son:

```text
Uvicorn
Daphne
Hypercorn
```

En una arquitectura ASGI podría utilizarse:

```text
Nginx
  ↓
Uvicorn
  ↓
Django ASGI
```

EcoRed utiliza en este taller una arquitectura convencional de solicitud/respuesta, por lo que **WSGI + Gunicorn** es suficiente y reduce la complejidad.

---

## Servidores de desarrollo frente a servidores de producción

Durante desarrollo pueden utilizarse:

```text
Vite Development Server :5173
Django runserver         :8000
```

Estos servidores priorizan:

- rapidez de desarrollo;
- recarga automática;
- mensajes detallados;
- facilidad de depuración.

No deben confundirse con una configuración de despliegue.

Django indica expresamente que `runserver` es un servidor ligero de desarrollo y no está diseñado para producción.

En este taller se pasa de:

```text
DESARROLLO

Vite :5173
Django runserver :8000
```

a:

```text
DESPLIEGUE

Nginx :10000
Gunicorn :8000
Django
```

---

## ¿Qué es Render?

**Render** es una plataforma administrada para desplegar y ejecutar aplicaciones y servicios.

Su propósito es abstraer buena parte de la infraestructura que, de otra forma, tendría que administrar el equipo:

- aprovisionamiento de capacidad de cómputo;
- exposición pública de la aplicación;
- dominio `onrender.com`;
- certificados TLS;
- health checks;
- variables y secretos;
- logs;
- ciclo de despliegue;
- reinicios y operación del servicio.

Para este taller Render **no construirá EcoRed desde GitHub**.

Recibirá directamente la imagen previamente construida y almacenada en Docker Hub:

```text
Docker Hub
    ↓
Render
    ↓
contenedor
```

---

## ¿Qué es un Web Service dentro de Render?

Un **Web Service** es un tipo de recurso de Render destinado a ejecutar una aplicación que debe recibir solicitudes HTTP desde Internet.

Render asigna al Web Service:

```text
https://nombre-del-servicio.onrender.com
```

y enruta el tráfico público al puerto HTTP en el que escucha la aplicación.

Render establece que un Web Service debe escuchar en:

```text
0.0.0.0:<puerto>
```

y utiliza `10000` como puerto esperado por defecto.

En este taller:

```text
Render HTTPS
     │
     ▼
Web Service
     │
     ▼
Nginx :10000
```

Un Web Service no es lo mismo que “Render completo”. Es **un recurso desplegable dentro de la plataforma Render**.

Render también dispone de otros tipos de recursos con responsabilidades diferentes, por ejemplo servicios privados o procesos sin entrada HTTP pública.

---

## Render frente a AWS, Microsoft Azure y Google Cloud

Render y los grandes proveedores de nube no se encuentran exactamente en el mismo nivel de abstracción.

**AWS, Microsoft Azure y Google Cloud son proveedores de nube de propósito general**. Ofrecen cientos de servicios de infraestructura y plataforma:

- máquinas virtuales;
- redes virtuales;
- balanceadores;
- almacenamiento;
- bases de datos;
- Kubernetes;
- IAM;
- serverless;
- analítica;
- IA;
- observabilidad;
- plataformas para contenedores.

Render se concentra mucho más en la **experiencia de desplegar y operar aplicaciones**, ocultando gran parte de esas decisiones.

Una comparación conceptual es:

| Plataforma | Servicio comparable para contenedores | Nivel de abstracción |
|---|---|---|
| **Render** | Web Service | Muy administrado; el desarrollador entrega código o imagen |
| **Azure** | Azure Container Apps | Plataforma serverless administrada para contenedores |
| **Google Cloud** | Cloud Run | Plataforma totalmente administrada para ejecutar contenedores y código |
| **AWS** | Amazon ECS sobre AWS Fargate | Contenedores sin administrar servidores, pero con mayor configuración explícita de red, tareas, IAM y servicios |

La equivalencia no es exacta.

Por ejemplo, en AWS ECS/Fargate normalmente se deben comprender elementos adicionales como:

```text
Task Definition
ECS Service
VPC
Security Groups
IAM Roles
Load Balancer
```

Mientras que en Render gran parte de ese trabajo queda abstraído mediante:

```text
Web Service
Image
Environment Variables
Secret Files
Health Check
```

Desde una perspectiva de modelos de servicio, Render se aproxima a una **plataforma administrada tipo PaaS**, mientras que AWS, Azure y Google Cloud permiten trabajar desde niveles de infraestructura muy bajos hasta plataformas administradas de alto nivel.

> Nota técnica: AWS App Runner fue durante varios años una alternativa aún más cercana conceptualmente a Render Web Services, pero desde el 30 de abril de 2026 AWS ya no lo ofrece a nuevos clientes. Para un análisis actual resulta más apropiado considerar ECS/Fargate dentro de AWS.

---

# Objetivo y arquitectura del taller

El objetivo es construir una sola imagen Docker de EcoRed y demostrar que el mismo artefacto puede ejecutarse en tres entornos:

1. localmente;
2. descargado desde Docker Hub;
3. como Web Service en Render.

El flujo es:

```text
Código EcoRed
     ↓
Dockerfile
     ↓
docker build
     ↓
Imagen
     ↓
Prueba local
     ↓
Docker Hub
     ↓
docker pull
     ↓
Nueva prueba local
     ↓
Render Existing Image
     ↓
Web Service público
```

**GitHub no participa en el despliegue de este taller.**

---

# Fase 1. Conceptos y preparación

## Paso 1.1. Identificar los componentes técnicos

| Término | Definición aplicada al taller |
|---|---|
| **Dockerfile** | Receta declarativa que define cómo construir una imagen. |
| **Imagen Docker** | Artefacto versionado e inmutable que contiene la aplicación y su runtime. |
| **Contenedor** | Instancia ejecutable creada a partir de una imagen. |
| **Docker Compose** | Herramienta para declarar mediante YAML cómo construir y ejecutar contenedores localmente. |
| **YAML** | Lenguaje de serialización legible utilizado para archivos de configuración. |
| **Docker Hub** | Container Registry utilizado para almacenar y distribuir la imagen. |
| **Render** | Plataforma administrada donde se ejecutará el contenedor. |
| **Web Service** | Recurso de Render accesible públicamente por HTTP/HTTPS. |
| **Nginx** | Servidor HTTP y reverse proxy. |
| **Gunicorn** | Servidor de aplicaciones WSGI para Python. |
| **Django** | Framework de backend que implementa la lógica y la API. |
| **MongoDB Atlas** | Backing Service para persistencia de datos. |
| **Firebase** | Backing Service utilizado, entre otras funciones, para autenticación. |

---

## Paso 1.2. Verificar el ambiente local

Comprobar Docker Desktop:

```powershell
docker version
docker compose version
```

Verificar archivos:

```powershell
Test-Path .\backend\.env
Test-Path .\backend\firebase-service-account.json
Test-Path .\frontend\.env
Test-Path .\backend\requirements.txt
Test-Path .\frontend\package.json
```

Todos deben responder:

```text
True
```

Los dos archivos privados:

```text
backend/.env
backend/firebase-service-account.json
```

se necesitan para ejecutar EcoRed, pero no deben quedar almacenados dentro de la imagen publicada.

---

## Paso 1.3. Preparar la infraestructura del contenedor

Este paso concentra la mayor parte del diseño técnico del taller.

La contenerización no consiste simplemente en “meter el código en Docker”. Es necesario decidir:

1. qué se construye;
2. qué queda dentro de la imagen;
3. qué configuración se suministra al ejecutar;
4. qué proceso recibe tráfico HTTP;
5. cómo el frontend se comunica con el backend;
6. cómo se protegen los secretos.

Los archivos serán:

```text
ecored-circular/
├── Dockerfile
├── compose.yaml
├── nginx.conf
├── start.sh
├── .dockerignore
├── backend/
└── frontend/
```

### 1.3.1. `.dockerignore`: controlar el contexto de construcción

Docker construye una imagen a partir de un **build context**.

El contexto es el conjunto de archivos que Docker puede utilizar durante:

```text
docker build
```

`.dockerignore` evita enviar al motor de construcción archivos innecesarios o sensibles.

```dockerignore
.git/
.vscode/
.idea/

**/venv/
**/.venv/
**/node_modules/
**/__pycache__/
**/*.pyc
**/*.pyo
**/dist/
**/build/

backend/.env
backend/.env.*
backend/firebase-service-account.json

*.zip
*.tar
*.tar.gz
*.log
```

La decisión de seguridad es:

```text
backend/.env                         ❌ fuera de la imagen
firebase-service-account.json       ❌ fuera de la imagen
frontend/.env                       ✅ disponible durante el build
```

¿Por qué `frontend/.env` sí participa?

Vite procesa variables `VITE_*` durante:

```text
npm run build
```

y genera JavaScript estático.

Por ello, si se excluye accidentalmente:

```text
frontend/.env
```

el frontend puede mostrar:

```text
Configuración incompleta
```

por falta de:

```text
VITE_FIREBASE_API_KEY
VITE_FIREBASE_AUTH_DOMAIN
VITE_FIREBASE_PROJECT_ID
VITE_FIREBASE_APP_ID
```

Las variables `VITE_*` deben considerarse **configuración pública del frontend**: cualquier valor usado por el navegador puede ser inspeccionado por el usuario. Nunca deben contener claves privadas del backend.

---

### 1.3.2. `nginx.conf`: diseñar el punto de entrada HTTP

```nginx
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;

    access_log /dev/stdout;
    error_log /dev/stderr warn;

    server {
        listen 10000;
        server_name _;

        root /usr/share/nginx/html;
        index index.html;

        location /api/ {
            proxy_pass http://127.0.0.1:8000;

            proxy_set_header Host localhost;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /assets/ {
            try_files $uri =404;
            add_header Cache-Control "public, max-age=31536000, immutable";
        }

        location = /index.html {
            add_header Cache-Control "no-cache, no-store, must-revalidate";
        }

        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```

Esta configuración crea dos rutas lógicas:

```text
/           → React
/api/       → Django
```

Por ejemplo:

```text
https://ecored-circular.onrender.com/api/health/
```

realiza:

```text
Render
  ↓
Nginx :10000
  ↓
proxy_pass
  ↓
Gunicorn :8000
  ↓
Django
```

La regla:

```nginx
try_files $uri $uri/ /index.html;
```

es importante para una SPA de React, porque rutas del navegador como:

```text
/home
```

deben regresar a `index.html` para que el router del frontend determine qué componente mostrar.

Los bloques `/assets/` e `index.html` aplican políticas de caché distintas. Los assets generados por Vite tienen nombres con hash y pueden almacenarse agresivamente; `index.html` debe renovarse para no apuntar a archivos antiguos después de un nuevo despliegue.

---

### 1.3.3. `start.sh`: establecer el ciclo de arranque

```sh
#!/bin/sh
set -e

if [ -f /etc/secrets/firebase-service-account.json ]; then
    ln -sf \
      /etc/secrets/firebase-service-account.json \
      /app/backend/firebase-service-account.json
fi

cd /app/backend

gunicorn config.wsgi:application \
    --bind 127.0.0.1:8000 \
    --workers 1 \
    --access-logfile - \
    --error-logfile - &

exec nginx -g "daemon off;"
```

El script ejecuta dos responsabilidades:

```text
Gunicorn
    └── aplicación Django vía WSGI

Nginx
    └── proceso HTTP frontal
```

Gunicorn se enlaza únicamente a:

```text
127.0.0.1:8000
```

Esto significa que no se publica directamente hacia Internet.

Solo Nginx es el punto de entrada externo del contenedor.

---

### 1.3.4. `Dockerfile`: construir un artefacto reproducible

```dockerfile
# syntax=docker/dockerfile:1

FROM node:24-bookworm-slim AS frontend-build

WORKDIR /frontend

COPY frontend/package.json frontend/package-lock.json ./
RUN npm ci

COPY frontend/ ./

ENV VITE_API_URL=/api

RUN npm run build


FROM python:3.13-slim-bookworm

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN apt-get update \
    && apt-get install -y --no-install-recommends nginx \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY backend/requirements.txt /tmp/requirements-utf16.txt

RUN python -c "from pathlib import Path; origen=Path('/tmp/requirements-utf16.txt'); destino=Path('/tmp/requirements.txt'); destino.write_text(origen.read_text(encoding='utf-16'), encoding='utf-8')"

RUN python -m pip install \
    --no-cache-dir \
    -r /tmp/requirements.txt

COPY backend/ /app/backend/

COPY --from=frontend-build \
    /frontend/dist \
    /usr/share/nginx/html

COPY nginx.conf /etc/nginx/nginx.conf
COPY start.sh /start.sh

RUN sed -i 's/\r$//' /start.sh \
    && chmod +x /start.sh \
    && nginx -t

EXPOSE 10000

HEALTHCHECK \
    --interval=30s \
    --timeout=5s \
    --start-period=30s \
    --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:10000/api/health/', timeout=3)"

CMD ["/start.sh"]
```

El Dockerfile utiliza un **multi-stage build**.

### Etapa de construcción

```text
Node
  ↓
npm ci
  ↓
npm run build
  ↓
dist/
```

### Etapa de ejecución

```text
Python
Django
Gunicorn
Nginx
React compilado
```

La ventaja es que la imagen final no necesita:

```text
npm
Node
node_modules
Vite Development Server
```

La instrucción:

```dockerfile
ENV VITE_API_URL=/api
```

evita fijar:

```text
localhost:8000
```

en el frontend final.

Así, la misma aplicación puede llamar:

```text
/api
```

independientemente de que se ejecute en:

```text
localhost
Render
otro dominio
```

---

### 1.3.5. `compose.yaml`: declarar la ejecución local

```yaml
name: ecored-circular

services:
  app:

    platform: linux/amd64

    build:
      context: .
      dockerfile: Dockerfile

    image: ecored-circular:v1.0
    container_name: ecored-circular

    ports:
      - "8080:10000"

    volumes:
      - ./backend/.env:/app/backend/.env:ro
      - ./backend/firebase-service-account.json:/app/backend/firebase-service-account.json:ro

    restart: unless-stopped
```

Compose define cómo **ejecutar** la imagen localmente.

El mapeo:

```text
8080:10000
```

significa:

```text
HOST Windows        CONTENEDOR
localhost:8080  →   Nginx:10000
```

Los `volumes` utilizados son realmente **bind mounts**: archivos existentes en Windows se presentan dentro del contenedor sin copiarlos dentro de la imagen.

Esto permite:

```text
imagen reutilizable
+
configuración privada suministrada en runtime
```

El modo:

```text
:ro
```

significa **read-only**.

El contenedor puede leer los secretos, pero no modificarlos.

---

# Fase 2. Construir y probar el contenedor localmente

## Paso 2.1. Construir e iniciar

Validar YAML:

```powershell
docker compose config --quiet
```

Construir:

```powershell
docker compose build --no-cache
```

`--no-cache` obliga a regenerar las capas. En este taller es especialmente útil porque garantiza que:

```text
frontend/.env
    ↓
npm run build
    ↓
React compilado actualizado
```

Iniciar:

```powershell
docker compose up -d
```

Verificar:

```powershell
docker ps
```

Debe aparecer:

```text
0.0.0.0:8080->10000/tcp
```

y:

```text
COMMAND
/start.sh
```

Consultar health:

```powershell
docker inspect --format="{{.State.Health.Status}}" ecored-circular
```

Después del periodo de inicio:

```text
healthy
```

---

## Paso 2.2. Realizar la prueba funcional mínima

Backend:

```powershell
curl.exe http://localhost:8080/api/health/
```

Resultado:

```json
{"status":"ok"}
```

Frontend:

```text
http://localhost:8080
```

Validar:

1. carga de EcoRed;
2. autenticación;
3. consulta de empresas;
4. consulta de materiales;
5. creación de un registro.

Después:

```powershell
docker compose restart
```

y repetir la prueba.

Finalmente reiniciar Windows y verificar:

```powershell
docker ps
```

La opción:

```yaml
restart: unless-stopped
```

permite que Docker vuelva a iniciar el contenedor cuando Docker Desktop se encuentre disponible.

---

# Fase 3. Publicar y validar la imagen desde Docker Hub

## Paso 3.1. Etiquetar y publicar

Crear en Docker Hub:

```text
ecored-circular
```

Iniciar sesión:

```powershell
docker login
```

Etiquetar:

```powershell
docker tag ecored-circular:v1.0 TU_USUARIO/ecored-circular:v1.0
```

La referencia:

```text
TU_USUARIO/ecored-circular:v1.0
```

se compone de:

```text
namespace / repository : tag
```

Publicar:

```powershell
docker push TU_USUARIO/ecored-circular:v1.0
```

Docker Hub actúa como **Container Registry**, es decir, almacena y distribuye imágenes versionadas.

---

## Paso 3.2. Eliminar, descargar y ejecutar la imagen publicada

Este paso es metodológicamente importante porque diferencia dos pruebas:

### Prueba 1

```text
“la imagen que construí en mi computador funciona”
```

### Prueba 2

```text
“el artefacto que publiqué en el registry puede recuperarse y ejecutarse”
```

La segunda prueba es la relevante para un despliegue Cloud.

### ¿Qué ocurre durante `docker push` y `docker pull`?

Una imagen Docker está formada por **capas**.

Conceptualmente:

```text
Imagen v1.0
├── capa sistema base
├── capa Nginx
├── capa dependencias Python
├── capa Django
├── capa React compilado
└── configuración de arranque
```

Durante:

```text
docker push
```

Docker calcula y publica las capas necesarias en el registry.

Durante:

```text
docker pull
```

Docker recupera esas capas y reconstruye localmente la referencia a la imagen.

Esta característica permite:

- reutilización de capas;
- distribución eficiente;
- identificación mediante hashes;
- reproducibilidad del artefacto.

### ¿Por qué eliminar primero la imagen local?

Si se ejecutara inmediatamente:

```powershell
docker run TU_USUARIO/ecored-circular:v1.0
```

sin eliminar la imagen, Docker podría utilizar la copia que ya estaba almacenada localmente.

Eso no demostraría que Docker Hub contiene correctamente el artefacto.

Por tanto, primero detener:

```powershell
docker compose down
```

y eliminar las referencias locales:

```powershell
docker image rm TU_USUARIO/ecored-circular:v1.0
docker image rm ecored-circular:v1.0
```

Después:

```powershell
docker images
```

y confirmar que EcoRed no aparece.

Ahora descargar:

```powershell
docker pull TU_USUARIO/ecored-circular:v1.0
```

En este momento la procedencia queda clara:

```text
Docker Hub
    ↓
docker pull
    ↓
Docker Engine local
    ↓
imagen v1.0
```

### Verificar que la imagen descargada corresponde a la arquitectura correcta

```powershell
docker image inspect TU_USUARIO/ecored-circular:v1.0 --format "{{json .Config.Cmd}}"
```

Resultado esperado:

```text
["/start.sh"]
```

Esto confirma que no se descargó accidentalmente una versión antigua que ejecutaba:

```text
npm run dev
python manage.py runserver
```

También:

```powershell
docker image inspect TU_USUARIO/ecored-circular:v1.0 --format "{{json .Config.ExposedPorts}}"
```

debe incluir:

```text
10000/tcp
```

### ¿Por qué los secretos deben volver a montarse?

Docker Hub almacena la imagen, pero deliberadamente no contiene:

```text
backend/.env
firebase-service-account.json
```

Por tanto, la imagen es portable pero todavía necesita recibir su configuración privada al ejecutarse.

Resolver las rutas:

```powershell
$BackendEnv = (Resolve-Path .\backend\.env).Path
$Firebase = (Resolve-Path .\backend\firebase-service-account.json).Path
```

Ejecutar:

```powershell
docker run -d `
  --name ecored-hub `
  --restart unless-stopped `
  -p 8080:10000 `
  --mount "type=bind,source=$BackendEnv,target=/app/backend/.env,readonly" `
  --mount "type=bind,source=$Firebase,target=/app/backend/firebase-service-account.json,readonly" `
  TU_USUARIO/ecored-circular:v1.0
```

Interpretación de los parámetros:

| Parámetro | Significado |
|---|---|
| `-d` | Ejecuta el contenedor en segundo plano. |
| `--name ecored-hub` | Asigna un nombre identificable al contenedor. |
| `--restart unless-stopped` | Reinicia el contenedor automáticamente salvo detención explícita. |
| `-p 8080:10000` | Mapea el puerto del host al puerto HTTP del contenedor. |
| `--mount ... .env` | Proporciona configuración privada al backend sin incorporarla a la imagen. |
| `--mount ... firebase...json` | Proporciona la credencial Firebase como archivo read-only. |
| `TU_USUARIO/...:v1.0` | Indica exactamente qué imagen versionada se debe ejecutar. |

Probar:

```powershell
curl.exe http://localhost:8080/api/health/
```

y:

```text
http://localhost:8080
```

Con esto se valida el flujo completo:

```text
Docker Hub
    ↓
imagen versionada
    ↓
docker pull
    ↓
docker run
    ↓
contenedor
    ↓
EcoRed
```

---

## Paso 3.3. Probar reinicio

Dejar `ecored-hub` ejecutándose y reiniciar Windows.

Después:

```powershell
docker ps
```

y probar:

```text
http://localhost:8080
```

Si funciona, el artefacto recuperado desde el registry ha superado la validación previa al despliegue Cloud.

---

# Fase 4. Desplegar la imagen en Render

## Paso 4.1. Crear el Web Service desde Docker Hub

En Render:

```text
New
→ Web Service
→ Existing Image
```

En:

```text
Image URL
```

ingresar:

```text
docker.io/TU_USUARIO/ecored-circular:v1.0
```

Si el repositorio es público:

```text
Credential
→ No credential
```

Después:

```text
Connect
```

Configurar:

```text
Name:
ecored-circular

Instance Type:
Free
```

Render creará un **image-backed Web Service**: el artefacto ejecutado procede de Docker Hub y no de un repositorio Git.

---

## Paso 4.2. Configurar runtime, secretos y health check

En:

```text
Environment Variables
→ Add from .env
```

copiar el contenido de:

```text
backend/.env
```

Revisar:

```env
PORT=10000
DJANGO_DEBUG=False
FIREBASE_CREDENTIALS_PATH=/etc/secrets/firebase-service-account.json
```

Mantener además:

```text
DJANGO_SECRET_KEY
MONGODB_URI
MONGODB_DB_NAME
```

Las variables:

```text
VITE_*
```

no se agregan en Render porque ya fueron procesadas durante el build de React.

### Secret File

En:

```text
Advanced
→ Secret Files
→ Add Secret File
```

crear:

```text
firebase-service-account.json
```

y copiar el contenido real del archivo local.

Render lo expone en:

```text
/etc/secrets/firebase-service-account.json
```

### Health Check

Cambiar:

```text
/healthz
```

por:

```text
/api/health/
```

Un **health check** es una comprobación automática que permite a la plataforma determinar si el servicio está respondiendo.

Dejar vacíos:

```text
Docker Command
Pre-Deploy Command
```

porque Docker ya define:

```dockerfile
CMD ["/start.sh"]
```

---

## Paso 4.3. Autorizar Render en Firebase y probar

Desplegar:

```text
Deploy Web Service
```

Supóngase que Render crea:

```text
https://ecored-circular.onrender.com
```

Firebase Authentication debe autorizar explícitamente ese dominio para OAuth.

Ir a:

```text
Firebase Console
→ Authentication
→ Settings
→ Authorized domains
→ Add domain
```

Agregar:

```text
ecored-circular.onrender.com
```

No escribir:

```text
https://
```

### ¿Por qué se necesita?

La autenticación con Google utiliza OAuth.

Operaciones como:

```text
signInWithPopup
signInWithRedirect
```

solo pueden ejecutarse desde dominios incluidos en la lista de dominios autorizados del proyecto Firebase.

Sin esta configuración el navegador puede indicar:

```text
The current domain is not authorized for OAuth operations
```

### Prueba final

Backend:

```text
https://ecored-circular.onrender.com/api/health/
```

Debe responder:

```json
{"status":"ok"}
```

Frontend:

```text
https://ecored-circular.onrender.com
```

Validar:

1. autenticación Google;
2. consulta;
3. creación de un registro;
4. actualización del navegador.

Si después de cambiar de versión el navegador reporta un archivo Vite antiguo como:

```text
/assets/HomePage-XXXX.js
```

con HTTP `404`, realizar primero:

```text
Ctrl + F5
```

o limpiar los datos del sitio.

---

# Fase 5. Pruebas finales y análisis Twelve-Factor

## Paso 5.1. Comprobar independencia del computador

Apagar completamente el computador utilizado para construir la imagen.

Desde otro equipo o teléfono abrir:

```text
https://ecored-circular.onrender.com
```

EcoRed debe continuar funcionando:

```text
PC local
   X

Docker Hub
   │
   ▼
Render Web Service
   │
   ▼
Contenedor
   │
   ├── MongoDB Atlas
   └── Firebase
```

Esto demuestra que el computador local ya no participa en la ejecución.

---

## Paso 5.2. Analizar Twelve-Factor

| Factor | Estado | Análisis | Cómo mejorarlo |
|---|---|---|---|
| **I. Codebase** | Cumple | Una base de código produce una imagen utilizada en distintos despliegues. | Mantener un único repositorio y versionado. |
| **II. Dependencies** | Cumple | Dependencias declaradas mediante `requirements.txt` y `package.json`. | Fijar versiones y normalizar `requirements.txt` a UTF-8. |
| **III. Config** | Parcial | Secretos del backend son externos, pero `VITE_*` queda compilado dentro del frontend. | Implementar configuración runtime del frontend. |
| **IV. Backing Services** | Cumple | MongoDB Atlas y Firebase permanecen fuera del contenedor. | Mantener conexiones completamente externalizadas. |
| **V. Build, Release, Run** | Cumple | Build local, release en Docker Hub y ejecución en Render están separados. | Utilizar tags inmutables por versión. |
| **VI. Processes** | Parcial | La persistencia está fuera del proceso, pero Nginx y Gunicorn comparten contenedor. | Separar responsabilidades en una arquitectura posterior. |
| **VII. Port Binding** | Cumple | Nginx publica el servicio mediante `10000`. | Hacer el puerto dinámico si se requiere mayor portabilidad. |
| **VIII. Concurrency** | No se demuestra | Se utiliza un contenedor y un worker Gunicorn. | Escalar workers y réplicas. |
| **IX. Disposability** | Parcial | El contenedor puede eliminarse y recrearse sin pérdida de datos. | Implementar shutdown coordinado de procesos. |
| **X. Dev/Prod Parity** | Parcial | Se utiliza la misma imagen, pero local y Render difieren en infraestructura y secretos. | Crear un staging equivalente a producción. |
| **XI. Logs** | Cumple | Nginx y Gunicorn escriben a stdout/stderr. | Centralizar observabilidad. |
| **XII. Admin Processes** | No se demuestra | No se ejecutan jobs administrativos one-off. | Separar migraciones y tareas administrativas. |

---

# Entregables

1. Evidencia de `docker ps` con EcoRed local.
2. Evidencia de `/api/health/`.
3. Prueba de funcionamiento después de reiniciar Windows.
4. Imagen `v1.0` publicada en Docker Hub.
5. Evidencia de eliminación, `docker pull` y ejecución de la imagen descargada.
6. Web Service de Render creado desde `Existing Image`.
7. Variables y Secret File configurados.
8. Dominio de Render agregado en Firebase Authorized Domains.
9. URL pública funcionando.
10. Login Google y una operación funcional.
11. Prueba con el computador local apagado.
12. Matriz Twelve-Factor.

---

# Referencias técnicas

- Python Software Foundation. **PEP 3333 – Python Web Server Gateway Interface v1.0.1**.  
  https://peps.python.org/pep-3333/

- Django Software Foundation. **How to deploy Django**.  
  https://docs.djangoproject.com/en/6.0/howto/deployment/

- Django Software Foundation. **How to deploy with WSGI**.  
  https://docs.djangoproject.com/en/6.0/howto/deployment/wsgi/

- F5 NGINX. **NGINX Reverse Proxy**.  
  https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/

- Render. **Web Services**.  
  https://render.com/docs/web-services

- Render. **Deploy a Prebuilt Docker Image**.  
  https://render.com/docs/deploying-an-image

- Microsoft. **Azure Container Apps**.  
  https://learn.microsoft.com/azure/container-apps/

- Google Cloud. **Cloud Run**.  
  https://cloud.google.com/run

- Amazon Web Services. **Amazon ECS with AWS Fargate**.  
  https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html

---

# Alcance

Este taller no busca implementar Kubernetes, microservicios, CI/CD ni una arquitectura empresarial completa.

Su finalidad es que el estudiante comprenda y pueda reproducir rigurosamente:

```text
servidor HTTP
      +
servidor de aplicación WSGI
      +
contenedor
      +
registry
      +
Web Service administrado
```

a través del flujo:

```text
EcoRed
  ↓
Docker
  ↓
Docker Hub
  ↓
Render
  ↓
Internet
```
