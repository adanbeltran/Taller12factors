# Requisitos
<img width="350" height="398" alt="image" src="https://github.com/user-attachments/assets/5a58c0a8-c004-46a0-925c-eab1e206a0d9" />


# Taller Basico EcoRed Circular en un solo contenedor sin modificar el código

> **Propósito:** ejecutar el backend Django y el frontend React/Vite dentro de un único contenedor, conservando el funcionamiento actual del proyecto.  
> **Nivel:** introductorio.  
> **Duración sugerida:** 2 a 3 horas.  
> **Condición:** no se modifica ningún archivo fuente de Django ni de React.  
> **Alcance:** laboratorio local. No corresponde a una solución de producción.

---

## 1. Resultado del taller

Al finalizar se tendrá:

```text
Navegador
   │
   ├── http://localhost:5173  ──► React + Vite
   │
   └── http://localhost:8000  ──► Django REST API
                                      │
                                      ├── MongoDB Atlas
                                      └── Firebase Admin
```

Dentro del contenedor se ejecutarán dos procesos:

```text
Contenedor ecored-circular
├── Django runserver     puerto 8000
└── Vite dev server     puerto 5173
```

El proyecto seguirá utilizando las mismas rutas actuales:

```text
Frontend: http://localhost:5173
Backend:  http://localhost:8000/api
```

---

## 2. ¿Por qué este taller no cambia el código?

El proyecto ya funciona localmente con:

```powershell
python manage.py runserver 8000
npm run dev
```

El taller reproduce esos mismos comandos dentro de Docker.

No se modifican:

- `settings.py`;
- `urls.py`;
- `mongo.py`;
- `firebase_auth.py`;
- componentes React;
- configuración de Vite;
- servicios HTTP;
- archivos `.env`.

Solamente se agregan cuatro archivos para Docker:

```text
ecored-circular/
├── Dockerfile
├── compose.yaml
├── .dockerignore
└── test-container.ps1
```

---

# Fase 1. Verificar el funcionamiento actual

## Paso 1. Ejecutar el backend localmente

Abra una terminal en la raíz del proyecto:

```powershell
# Entra en la carpeta del backend.
cd .\backend\

# Inicia Django en el puerto usado actualmente por el proyecto.
python manage.py runserver 8000
```

Compruebe:

```text
http://localhost:8000/api/health/
```

Respuesta esperada:

```json
{"status":"ok"}
```

## Paso 2. Ejecutar el frontend localmente

Abra otra terminal:

```powershell
# Entra en la carpeta del frontend.
cd .\frontend\

# Inicia el servidor de desarrollo de Vite.
npm run dev
```

Compruebe:

```text
http://localhost:5173
```

Realice una prueba mínima:

1. iniciar sesión;
2. abrir el módulo de empresas;
3. abrir el módulo de materiales;
4. consultar los registros existentes.

Después, detenga ambos procesos con:

```text
Ctrl + C
```

### ¿Por qué se realiza esta prueba?

Permite confirmar que el código, MongoDB Atlas y Firebase funcionan antes de introducir Docker.

### ¿Qué ocurre si se omite?

Si el contenedor falla, no será posible saber si el problema pertenece al proyecto original o a la contenerización.

---

# Fase 2. Preparar el contexto de Docker

## Paso 3. Crear `.dockerignore`

Cree el archivo `.dockerignore` en la raíz del proyecto:

```dockerignore
# No envía el repositorio Git al proceso de construcción.
.git/

# No envía configuraciones locales de editores.
.vscode/
.idea/

# No copia el entorno virtual creado en Windows.
**/venv/
**/.venv/

# No copia dependencias Node instaladas en el computador.
**/node_modules/

# No copia caché ni bytecode Python.
**/__pycache__/
**/*.pyc
**/*.pyo

# No copia compilaciones anteriores.
**/dist/
**/build/

# No incorpora archivos de variables de entorno en la imagen.
**/.env
**/.env.*

# Permite conservar las plantillas sin valores reales.
!**/.env.example

# No incorpora la cuenta de servicio Firebase en la imagen.
backend/firebase-service-account.json

# No envía archivos comprimidos.
*.zip
*.tar
*.tar.gz

# No envía logs ni pruebas HTTP manuales.
**/*.log
**/*.http
```

## Explicación

| Patrón | Motivo |
|---|---|
| `.git/` | El historial no es necesario para ejecutar la aplicación. |
| `venv/` | El entorno de Windows no funciona correctamente dentro de Linux. |
| `node_modules/` | Docker instalará dependencias compatibles con Linux. |
| `.env` | Evita copiar claves y contraseñas dentro de la imagen. |
| `firebase-service-account.json` | Evita incorporar una credencial privada. |
| `dist/` | Este taller utiliza Vite en modo desarrollo y no necesita un build previo. |

### ¿Por qué se crea?

Docker envía al constructor todos los archivos que no estén excluidos. El archivo reduce el tamaño del contexto y evita copiar secretos accidentalmente.

### ¿Qué ocurre si no se crea?

La imagen podría incluir:

- archivos `.env`;
- la cuenta de servicio Firebase;
- `node_modules`;
- el entorno virtual de Windows;
- el historial Git;
- archivos innecesarios.

---

# Fase 3. Crear la imagen

## Paso 4. Crear `Dockerfile`

Cree `Dockerfile` en la raíz:

```dockerfile
# Activa la sintaxis actual del Dockerfile.
# syntax=docker/dockerfile:1

# ------------------------------------------------------------
# ETAPA 1: obtener Node.js y npm
# ------------------------------------------------------------

# Usa la imagen oficial de Node 24 como fuente del runtime Node.
FROM node:24-bookworm-slim AS node-runtime


# ------------------------------------------------------------
# ETAPA 2: imagen final con Python y Node
# ------------------------------------------------------------

# Usa Python 3.13 como base principal del contenedor.
FROM python:3.13-slim-bookworm

# Evita que Python cree archivos .pyc.
ENV PYTHONDONTWRITEBYTECODE=1

# Envía inmediatamente los mensajes Python a la consola.
ENV PYTHONUNBUFFERED=1

# Instala la biblioteca requerida por el ejecutable de Node.
RUN apt-get update \
    && apt-get install -y --no-install-recommends libstdc++6 \
    && rm -rf /var/lib/apt/lists/*

# Copia el ejecutable Node desde la primera etapa.
COPY --from=node-runtime /usr/local/bin/node /usr/local/bin/node

# Copia npm y sus módulos globales desde la primera etapa.
COPY --from=node-runtime /usr/local/lib/node_modules /usr/local/lib/node_modules

# Crea el comando npm apuntando al archivo principal de npm.
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

# Define la carpeta general de la aplicación.
WORKDIR /app

# Copia temporalmente requirements.txt.
COPY backend/requirements.txt /tmp/requirements-utf16.txt

# Convierte una copia de UTF-16 a UTF-8 sin modificar el archivo original.
RUN python -c "from pathlib import Path; origen=Path('/tmp/requirements-utf16.txt'); destino=Path('/tmp/requirements.txt'); destino.write_text(origen.read_text(encoding='utf-16'), encoding='utf-8')"

# Instala las dependencias Python dentro de la imagen.
RUN python -m pip install \
    --no-cache-dir \
    -r /tmp/requirements.txt

# Crea la carpeta del frontend.
WORKDIR /app/frontend

# Copia primero los manifiestos de Node.
COPY frontend/package.json frontend/package-lock.json ./

# Instala exactamente las versiones registradas en package-lock.json.
RUN npm ci

# Regresa a la carpeta general.
WORKDIR /app

# Copia el backend sin .env ni la cuenta de servicio.
COPY backend/ /app/backend/

# Copia el frontend sin .env ni node_modules locales.
COPY frontend/ /app/frontend/

# Documenta el puerto de Django.
EXPOSE 8000

# Documenta el puerto de Vite.
EXPOSE 5173

# Verifica que Django y Vite respondan.
HEALTHCHECK \
    --interval=20s \
    --timeout=5s \
    --start-period=30s \
    --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/api/health/', timeout=3); urllib.request.urlopen('http://127.0.0.1:5173/', timeout=3)" || exit 1

# Inicia Django y Vite al crear el contenedor.
CMD ["sh", "-c", "(cd /app/backend && python manage.py runserver 0.0.0.0:8000 --noreload) & (cd /app/frontend && npm run dev -- --host 0.0.0.0) & wait"]
```

---

## 5. Explicación del Dockerfile

### `FROM node:24-bookworm-slim AS node-runtime`

Proporciona Node.js y npm.

**Si se omite:** Vite no podrá ejecutarse.

### `FROM python:3.13-slim-bookworm`

Define Python como base final.

**Si se omite:** Django no podrá ejecutarse.

### `ENV PYTHONDONTWRITEBYTECODE=1`

Evita crear bytecode Python dentro del contenedor.

**Si se omite:** la aplicación puede funcionar, pero se crearán archivos innecesarios.

### `ENV PYTHONUNBUFFERED=1`

Envía los mensajes Python directamente a la salida del contenedor.

**Si se omite:** algunos mensajes podrían aparecer con retraso.

### Instalación de `libstdc++6`

Proporciona una biblioteca requerida por Node.

**Si se omite:** el ejecutable de Node podría no iniciar.

### Copias desde `node-runtime`

Copian Node y npm a la imagen Python.

**Si se omiten:** la imagen tendría Python, pero no podría iniciar Vite.

### Conversión de `requirements.txt`

El archivo original está en UTF-16. Docker crea una copia UTF-8 dentro de la imagen.

**Si se omite:** `pip` puede fallar al interpretar el archivo.

**Importante:** el archivo del proyecto no se modifica.

### `pip install`

Instala Django, Django REST Framework, PyMongo y Firebase Admin.

**Si se omite:** el backend fallará por módulos no encontrados.

### `npm ci`

Instala las versiones exactas del archivo `package-lock.json`.

**Si se omite:** Vite y React no estarán disponibles.

### `COPY backend/` y `COPY frontend/`

Copian el código existente.

**Si se omiten:** el contenedor no tendrá la aplicación.

### `EXPOSE`

Documenta los dos puertos internos.

**Si se omite:** la aplicación todavía podría funcionar con `ports`, pero la imagen no documentaría correctamente sus interfaces.

### `HEALTHCHECK`

Comprueba los dos servidores.

**Si se omite:** Docker solo sabrá que el contenedor sigue iniciado, pero no que las aplicaciones responden.

### `CMD`

Inicia Django y Vite simultáneamente.

**Si se omite:** el contenedor terminará inmediatamente porque no tendrá un proceso principal.

---

# Fase 4. Configurar la ejecución

## Paso 5. Crear `compose.yaml`

Cree `compose.yaml` en la raíz:

```yaml
# Define un nombre estable para el proyecto.
name: ecored-circular-previo

# Declara los servicios del laboratorio.
services:

  # Único servicio del taller.
  app:

    # Construye la imagen desde la raíz.
    build:

      # Define el contexto de construcción.
      context: .

      # Selecciona el Dockerfile.
      dockerfile: Dockerfile

    # Asigna un nombre a la imagen resultante.
    image: ecored-circular:previo

    # Publica los dos puertos actuales del proyecto.
    ports:

      # Permite abrir Django desde el computador.
      - "8000:8000"

      # Permite abrir Vite desde el computador.
      - "5173:5173"

    # Monta la configuración sin copiarla dentro de la imagen.
    volumes:

      # Monta las variables privadas del backend como solo lectura.
      - ./backend/.env:/app/backend/.env:ro

      # Monta la cuenta de servicio Firebase como solo lectura.
      - ./backend/firebase-service-account.json:/app/backend/firebase-service-account.json:ro

      # Monta la configuración pública utilizada por Vite.
      - ./frontend/.env:/app/frontend/.env:ro
```

---

## 6. ¿Qué hacen los montajes?

### Backend `.env`

```yaml
- ./backend/.env:/app/backend/.env:ro
```

- el archivo permanece en el computador;
- no se copia a la imagen;
- aparece dentro del contenedor al iniciarlo;
- `:ro` significa solo lectura.

### Cuenta de servicio Firebase

```yaml
- ./backend/firebase-service-account.json:/app/backend/firebase-service-account.json:ro
```

- no se incorpora en la imagen;
- conserva la ruta esperada por el código;
- el backend puede leerla;
- el contenedor no puede modificarla.

### Frontend `.env`

```yaml
- ./frontend/.env:/app/frontend/.env:ro
```

- no se copia a la imagen;
- Vite lo lee cuando inicia;
- sus valores `VITE_*` se entregan al navegador;
- no debe contener secretos del servidor.

### ¿Por qué se usan montajes?

Permiten conservar el código y la configuración actuales sin incorporarlos dentro de la imagen.

### ¿Qué ocurre si no se montan?

- Django no encontrará sus variables;
- Firebase no encontrará la cuenta de servicio;
- Vite informará que faltan variables obligatorias;
- la aplicación no iniciará correctamente.

---

# Fase 5. Construir y ejecutar

## Paso 6. Verificar archivos requeridos

Desde la raíz:

```powershell
# Comprueba el archivo de variables del backend.
Test-Path .\backend\.env

# Comprueba la cuenta de servicio Firebase.
Test-Path .\backend\firebase-service-account.json

# Comprueba el archivo de variables del frontend.
Test-Path .\frontend\.env
```

Los tres comandos deben devolver:

```text
True
```

## Paso 7. Verificar Docker

```powershell
# Comprueba cliente y motor Docker.
docker version

# Comprueba Docker Compose.
docker compose version
```

## Paso 8. Validar Compose

```powershell
# Valida la sintaxis sin imprimir la configuración completa.
docker compose -f compose.yaml config --quiet
```

### ¿Por qué se utiliza `--quiet`?

Evita mostrar información innecesaria y confirma únicamente si el YAML es válido.

## Paso 9. Construir la imagen

```powershell
# Construye la imagen definida en Compose.
docker compose -f compose.yaml build
```

Durante el proceso deben aparecer:

```text
pip install
npm ci
```

## Paso 10. Iniciar el contenedor

```powershell
# Crea e inicia el contenedor en segundo plano.
docker compose -f compose.yaml up -d
```

## Paso 11. Consultar el estado

```powershell
# Muestra el estado del servicio.
docker compose -f compose.yaml ps
```

Después del periodo de inicio se espera:

```text
running (healthy)
```

---

# Fase 6. Pruebas

## Prueba 1. Backend

```powershell
# Consulta el endpoint de salud de Django.
curl.exe http://localhost:8000/api/health/
```

Resultado esperado:

```json
{"status":"ok"}
```

## Prueba 2. Frontend

```powershell
# Solicita la página principal de Vite.
curl.exe -I http://localhost:5173/
```

Resultado esperado:

```text
HTTP/1.1 200 OK
```

## Prueba 3. Navegador

Abra:

```text
http://localhost:5173
```

Compruebe:

1. aparece la pantalla de inicio o inicio de sesión;
2. puede autenticarse mediante Firebase;
3. puede consultar empresas;
4. puede registrar una empresa;
5. puede consultar materiales;
6. puede registrar un material.

## Prueba 4. Estado de salud de Docker

```powershell
# Obtiene el identificador del contenedor.
$containerId = docker compose -f compose.yaml ps -q app

# Consulta el resultado del HEALTHCHECK.
docker inspect `
  --format='{{.State.Health.Status}}' `
  $containerId
```

Resultado esperado:

```text
healthy
```

## Prueba 5. Confirmar que los secretos no están en la imagen

```powershell
# Crea un contenedor temporal para revisar la imagen.
docker run `
  --rm `
  --entrypoint sh `
  ecored-circular:previo `
  -c `
  "test ! -e /app/backend/.env && \
   test ! -e /app/frontend/.env && \
   test ! -e /app/backend/firebase-service-account.json && \
   echo OK"
```

Resultado esperado:

```text
OK
```

## Prueba 6. Confirmar que los archivos sí están montados

```powershell
# Comprueba los archivos requeridos dentro del contenedor activo.
docker compose `
  -f compose.yaml `
  exec `
  app `
  sh -c `
  "test -r /app/backend/.env && \
   test -r /app/frontend/.env && \
   test -r /app/backend/firebase-service-account.json && \
   echo OK"
```

Resultado esperado:

```text
OK
```

## Prueba 7. Confirmar los montajes

```powershell
# Muestra los montajes sin mostrar el contenido de los archivos.
docker inspect `
  --format='{{json .Mounts}}' `
  $containerId
```

Deben aparecer los destinos:

```text
/app/backend/.env
/app/frontend/.env
/app/backend/firebase-service-account.json
```

## Prueba 8. Reinicio

```powershell
# Reinicia el servicio.
docker compose -f compose.yaml restart app

# Espera unos segundos y vuelve a probar el backend.
curl.exe http://localhost:8000/api/health/
```

Los datos registrados deben continuar disponibles porque permanecen en MongoDB Atlas.

---

# Fase 7. Prueba automatizada

## Paso 12. Crear `test-container.ps1`

```powershell
# Detiene el script ante cualquier error.
$ErrorActionPreference = "Stop"

# Define la URL del backend.
$backendUrl = "http://localhost:8000"

# Define la URL del frontend.
$frontendUrl = "http://localhost:5173"


# Informa la primera prueba.
Write-Host "1. Probando Django..."

# Consulta el endpoint de salud.
$health = Invoke-RestMethod "$backendUrl/api/health/"

# Valida el contenido de la respuesta.
if ($health.status -ne "ok") {
    # Detiene el script si Django no responde correctamente.
    throw "Django no respondió status=ok."
}


# Informa la segunda prueba.
Write-Host "2. Probando Vite..."

# Consulta la página principal.
$frontend = Invoke-WebRequest `
    "$frontendUrl/" `
    -UseBasicParsing

# Valida el código HTTP.
if ($frontend.StatusCode -ne 200) {
    # Detiene el script si el frontend no responde.
    throw "Vite no respondió HTTP 200."
}


# Informa la consulta del contenedor.
Write-Host "3. Consultando el contenedor..."

# Obtiene el ID del servicio app.
$containerId = (
    docker compose `
      -f compose.yaml `
      ps -q app
).Trim()

# Comprueba que exista.
if (-not $containerId) {
    # Detiene el script si no hay contenedor.
    throw "No se encontró el contenedor."
}


# Informa la prueba del estado de salud.
Write-Host "4. Validando HEALTHCHECK..."

# Consulta la salud calculada por Docker.
$containerHealth = (
    docker inspect `
      --format='{{.State.Health.Status}}' `
      $containerId
).Trim()

# Exige el estado healthy.
if ($containerHealth -ne "healthy") {
    # Detiene el script e informa el estado real.
    throw "Estado del contenedor: $containerHealth"
}


# Informa la revisión de la imagen.
Write-Host "5. Revisando archivos sensibles..."

# Comprueba que los archivos privados no estén en la imagen.
docker run `
  --rm `
  --entrypoint sh `
  ecored-circular:previo `
  -c `
  "test ! -e /app/backend/.env && \
   test ! -e /app/frontend/.env && \
   test ! -e /app/backend/firebase-service-account.json"

# Comprueba el código de salida.
if ($LASTEXITCODE -ne 0) {
    # Detiene el script si encuentra un archivo privado.
    throw "La imagen contiene un archivo privado."
}


# Informa la revisión de montajes.
Write-Host "6. Revisando montajes..."

# Comprueba que los archivos estén disponibles en runtime.
docker compose `
  -f compose.yaml `
  exec `
  -T `
  app `
  sh -c `
  "test -r /app/backend/.env && \
   test -r /app/frontend/.env && \
   test -r /app/backend/firebase-service-account.json"

# Comprueba el código de salida.
if ($LASTEXITCODE -ne 0) {
    # Detiene el script si falta un montaje.
    throw "Falta uno de los archivos montados."
}


# Informa el resultado final.
Write-Host `
  "PRUEBAS DEL TALLER PREVIO SUPERADAS" `
  -ForegroundColor Green
```

Ejecute:

```powershell
# Ejecuta todas las pruebas básicas.
.\test-container.ps1
```

Resultado esperado:

```text
PRUEBAS DEL TALLER PREVIO SUPERADAS
```

---

# Fase 8. Detener el taller

```powershell
# Detiene y elimina el contenedor y la red.
docker compose -f compose.yaml down
```

Para eliminar también la imagen:

```powershell
# Elimina la imagen creada.
docker image rm ecored-circular:previo
```

---

# 9. ¿Qué queda y qué no queda en la imagen?

## Queda en la imagen

- código fuente Django;
- código fuente React;
- dependencias Python;
- `node_modules` instalados dentro de Linux;
- Node.js y npm;
- servidor de desarrollo Vite;
- comando para iniciar ambos procesos;
- configuración no sensible del Dockerfile.

## No queda en la imagen

- `backend/.env`;
- `frontend/.env`;
- cuenta de servicio Firebase;
- entorno virtual de Windows;
- `node_modules` del computador;
- historial Git;
- datos almacenados en MongoDB Atlas.

## Está disponible dentro del contenedor

Mediante montajes:

- `backend/.env`;
- `frontend/.env`;
- `firebase-service-account.json`.

Estos archivos no pertenecen a la imagen, pero el proceso del contenedor puede leerlos.

## Llega al navegador

Las variables cuyo nombre empieza por `VITE_` son utilizadas por el frontend y pueden ser inspeccionadas desde el navegador.

Por tanto, `frontend/.env` solo debe contener configuración pública:

```env
VITE_API_URL=http://localhost:8000/api
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_APP_ID=...
```

Nunca debe contener:

```env
VITE_MONGODB_URI=...
VITE_DJANGO_SECRET_KEY=...
VITE_FIREBASE_PRIVATE_KEY=...
VITE_DATABASE_PASSWORD=...
```

---

# 10. Temas de seguridad que deben quedar claros

## 10.1 Los montajes evitan copiar secretos a la imagen

Esto mejora la situación frente a utilizar:

```dockerfile
COPY backend/.env /app/backend/.env
```

Sin embargo, los archivos privados siguen existiendo en el computador anfitrión.

Una persona con acceso administrativo al host o al contenedor puede leerlos.

## 10.2 El código actual imprime la URI de MongoDB

El archivo actual `backend/companies/mongo.py` contiene una impresión de `MONGODB_URI`.

Por esa razón, la URI puede aparecer en la salida del contenedor.

Durante este taller:

- no publique capturas de logs;
- no comparta la salida completa;
- use una credencial exclusiva de laboratorio;
- cambie la contraseña después de una exposición.

La corrección apropiada requiere modificar el código y eliminar esa impresión. Esto se realizará en un taller posterior.

## 10.3 Firebase tiene configuración pública y credencial privada

La configuración `VITE_FIREBASE_*` utilizada por la aplicación web es visible para el navegador.

La cuenta:

```text
firebase-service-account.json
```

es privada y nunca debe:

- incluirse en Git;
- copiarse a una imagen;
- enviarse a estudiantes con valores reales;
- aparecer en capturas;
- compartirse por correo o mensajería.

## 10.4 `DEBUG=True` no es apropiado para producción

El proyecto actual puede tener:

```env
DJANGO_DEBUG=True
```

Esto es útil para desarrollo, pero puede mostrar información interna cuando ocurre un error.

## 10.5 No hay cifrado TLS

El laboratorio utiliza:

```text
http://localhost
```

No utiliza HTTPS. Es aceptable únicamente en una práctica local controlada.

## 10.6 La imagen contiene todo el código

Cualquier persona que reciba la imagen puede extraer:

- backend;
- frontend;
- estructura del proyecto;
- dependencias;
- rutas internas.

Docker empaqueta el código, pero no lo convierte en secreto.

---

# 11. Limitaciones técnicas del taller previo

## 11.1 Utiliza servidores de desarrollo

Se ejecutan:

```text
Django runserver
Vite dev server
```

No están destinados a operación productiva.

En un taller posterior deben reemplazarse por:

```text
Gunicorn
Frontend compilado
Nginx o WhiteNoise
```

## 11.2 Ejecuta dos procesos en un contenedor

El contenedor inicia Django y Vite mediante un comando de shell.

Limitaciones:

- administración básica de señales;
- cierre no completamente controlado;
- si un proceso falla, el otro puede permanecer temporalmente activo;
- no existe supervisión especializada.

## 11.3 Publica dos puertos

El usuario debe acceder a:

```text
5173 para frontend
8000 para backend
```

No existe un único punto de entrada HTTP.

## 11.4 Mantiene CORS

Frontend y backend utilizan orígenes diferentes:

```text
http://localhost:5173
http://localhost:8000
```

Django debe permitir explícitamente el origen del frontend.

## 11.5 La imagen es más grande

La imagen final contiene simultáneamente:

- Python;
- Node;
- npm;
- código fuente;
- dependencias Python;
- `node_modules`.

## 11.6 No usa un usuario no privilegiado

El contenedor se ejecuta con el usuario predeterminado de la imagen.

Una versión fortalecida debe crear y utilizar un usuario sin privilegios.

## 11.7 No usa filesystem de solo lectura

Los procesos pueden escribir en la capa temporal del contenedor.

## 11.8 No utiliza un gestor de secretos

Los secretos se montan desde archivos locales.

En producción deberían obtenerse desde una solución especializada.

## 11.9 Frontend y backend no pueden escalar independientemente

Ambos procesos forman una sola unidad de despliegue.

## 11.10 No es una arquitectura de microservicios

La presencia de Docker y una API no convierte la aplicación en microservicios.

---

# 12. Siguiente taller recomendado

Después de completar esta práctica, el siguiente taller debe:

1. eliminar la impresión de `MONGODB_URI`;
2. compilar React con `npm run build`;
3. reemplazar Vite por archivos estáticos;
4. reemplazar `runserver` por Gunicorn;
5. servir el frontend con Nginx o WhiteNoise;
6. publicar un único puerto;
7. utilizar secretos montados;
8. ejecutar con usuario no root;
9. agregar cierre controlado;
10. analizar qué variables quedan en la imagen, el contenedor y el navegador.

---

# 13. Entregables

```text
apellido-nombre-taller-previo/
├── Dockerfile
├── compose.yaml
├── .dockerignore
├── test-container.ps1
└── evidencias/
    ├── 01-build.png
    ├── 02-container-healthy.png
    ├── 03-backend-health.png
    ├── 04-frontend.png
    ├── 05-login.png
    ├── 06-materials.png
    ├── 07-no-secrets-image.png
    └── 08-tests-passed.png
```

No se entregan:

```text
backend/.env
frontend/.env
backend/firebase-service-account.json
tokens
contraseñas
URI reales
claves privadas
```

---

# 14. Criterios de evaluación

| Criterio | Porcentaje |
|---|---:|
| Construcción correcta de la imagen | 25 % |
| Ejecución de Django y Vite | 25 % |
| Pruebas funcionales | 25 % |
| Exclusión y montaje de archivos privados | 15 % |
| Análisis de seguridad y limitaciones | 10 % |
