# C. Creación de backend

[← B. Configuración de backing services](B-backing-services.md) | [Volver al README](../README.md) | [Siguiente: D. Creación de frontend →](D-frontend.md)

## Propósito de esta sección

En esta sección se construirá el backend del proyecto usando **Visual Studio Code** como entorno principal de trabajo. Primero se ejecutarán en la **terminal integrada de VS Code** los comandos necesarios para preparar la base del backend. Después, desde el **editor de VS Code**, se crearán los archivos de configuración y código. Finalmente, se volverá a la terminal para validar la ejecución del servicio.

**Se asume que ya el estudiante ha tiene la cadena de conexion a MongoDB, y las credenciales en firebase**

En coherencia con el alcance del taller, la app principal del backend será **`companies`**, ya que el foco funcional está en el **módulo de empresas** del caso de estudio.

---

## 1. Crear la estructura del proyecto e inicializar Git

En este paso se crea la estructura inicial del proyecto y se inicializa el repositorio Git.

Abrir **Visual Studio Code**, abrir una carpeta de trabajo vacía y luego abrir la terminal integrada desde:

- **Terminal > New Terminal**

Ejecutar:

```powershell
mkdir ecored-circular
cd ecored-circular
mkdir backend, frontend, docs
git init
git checkout -b develop
```

### Qué realiza este paso

- crea la carpeta raíz del proyecto,
- crea las carpetas principales `backend`, `frontend` y `docs`,
- inicializa Git,
- crea la rama de trabajo `develop`.

### Explicación de los comandos

- `mkdir ecored-circular` crea la carpeta principal del proyecto.
- `cd ecored-circular` entra a esa carpeta para trabajar dentro de ella.
- `mkdir backend, frontend, docs` crea las tres carpetas base:
  - `backend` para el servicio Django,
  - `frontend` para la interfaz React,
  - `docs` para la documentación del taller.
- `git init` inicializa el repositorio.
- `git checkout -b develop` crea y activa la rama de trabajo `develop`.

### Aclaración sobre proyecto, versión y ambiente

Es importante diferenciar:

- **proyecto**: `ecored-circular`, que corresponde a la única codebase del taller;
- **versión**: estado liberado del código, por ejemplo `v1.0`, `v1.0.1` o `v2.0`;
- **ambiente**: contexto donde una versión se ejecuta, por ejemplo `dev`, `test` o `prod`.

En una situación real puede ocurrir simultáneamente que:

- en **producción** se esté ejecutando una versión estable, por ejemplo `v1.0`,
- en **pruebas** se esté validando otra versión identificada, por ejemplo `v1.1`,
- en **desarrollo** el equipo continúe construyendo nuevas funcionalidades para una versión posterior, por ejemplo `v2.0`.

Esto no implica proyectos distintos. Sigue existiendo una sola codebase, pero con diferentes versiones liberadas ejecutándose en distintos ambientes según su nivel de madurez.

### Estructura esperada en este punto



```text
ecored-circular/
├── backend/
├── frontend/
├── docs/
```

### Aplicación de Twelve-Factor

- **Factor I. Codebase**: un solo repositorio y una sola base de código.

---

## 2. Crear ambiente virtual e instalar librerías

En este paso se prepara el entorno de ejecución del backend y se instalan las dependencias necesarias.

Ubicados en la terminal integrada, ejecutar:

```powershell
cd backend
python -m venv venv
venv\Scripts\activate
pip install django djangorestframework django-cors-headers python-dotenv firebase-admin pymongo gunicorn
pip freeze > requirements.txt
```

### Qué realiza este paso

- entra a la carpeta del backend,
- crea el entorno virtual,
- activa ese entorno,
- instala dependencias,
- genera `requirements.txt`.

### Explicación de los comandos

- `cd backend` mueve la terminal a la carpeta donde se construirá el servicio Django.
- `python -m venv venv` crea un entorno virtual aislado llamado `venv`.
- `venv\Scripts\activate` activa el entorno virtual en Windows.
- `pip install ...` instala los paquetes del backend:
  - `django`: framework principal,
  - `djangorestframework`: construcción de la API REST,
  - `django-cors-headers`: manejo de CORS,
  - `python-dotenv`: lectura de variables desde `.env`,
  - `firebase-admin`: validación de tokens de Firebase,
  - `pymongo`: conexión con MongoDB,
  - `gunicorn`: referencia de servidor WSGI para escenarios de despliegue.
- `pip freeze > requirements.txt` guarda las dependencias instaladas con sus versiones exactas.

### Aplicación de Twelve-Factor

- **Factor II. Dependencies**: dependencias declaradas explícitamente en `requirements.txt`.

---

## 3. Crear proyecto Django y app `companies`

En este paso se genera la base del backend con Django y se crea la app `companies`, que representa el módulo principal de este taller.

Todavía en la terminal integrada, ejecutar:

```powershell
django-admin startproject config .
python manage.py startapp companies
mkdir companies\management
mkdir companies\management\commands
```

### Qué realiza este paso

- crea el proyecto Django `config`,
- crea la app `companies`,
- prepara la estructura para comandos administrativos.

### Explicación de los comandos

- `django-admin startproject config .` crea el proyecto Django en la carpeta actual.
- `python manage.py startapp companies` crea la app que encapsulará la lógica del módulo de empresas.
- `mkdir companies\management` crea la carpeta base para comandos personalizados.
- `mkdir companies\management\commands` crea la ruta donde vivirá `seed_demo`.

### Estructura esperada en este punto


<img width="384" height="482" alt="image" src="https://github.com/user-attachments/assets/cea7c65d-d8b5-4a81-af0e-7745384a72a0" />


### Aplicación de Twelve-Factor

- **Factor XII. Admin processes**: queda preparada la ruta para procesos administrativos.

---

## 4. Crear archivos de configuración

En este paso se crean los archivos base de configuración del backend.

A partir de este punto, abrir en VS Code la carpeta `ecored-circular` y crear desde el panel **Explorer** los siguientes archivos:

- `.gitignore`
- `backend/.env.example`
- `backend/.env`
- `backend/firebase-service-account.json`


### Archivo `.gitignore`

```gitignore
backend/venv/
backend/.env
backend/firebase-service-account.json
frontend/.env
__pycache__/
*.pyc
node_modules/
dist/
build/
```

### Archivo `backend/.env.example`

```env
DJANGO_SECRET_KEY=
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
APP_ENV=dev
PORT=8000

MONGODB_URI=
MONGODB_DB_NAME=ecored_circular_db

CORS_ALLOWED_ORIGINS=http://localhost:5173

FIREBASE_CREDENTIALS_PATH=./firebase-service-account.json
```

Luego crear `backend/.env` copiando la misma estructura y completando los valores reales.

### Archivo `backend/config/settings.py`

```python
from pathlib import Path
from dotenv import load_dotenv
import os

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")

SECRET_KEY = os.getenv("DJANGO_SECRET_KEY")
DEBUG = os.getenv("DJANGO_DEBUG", "False") == "True"
ALLOWED_HOSTS = os.getenv("DJANGO_ALLOWED_HOSTS", "127.0.0.1,localhost").split(",")

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "rest_framework",
    "corsheaders",
    "companies",
]

MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]

ROOT_URLCONF = "config.urls"
WSGI_APPLICATION = "config.wsgi.application"

CORS_ALLOWED_ORIGINS = os.getenv("CORS_ALLOWED_ORIGINS", "http://localhost:5173").split(",")

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "companies.firebase_auth.FirebaseAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticatedOrReadOnly",
    ],
}

LANGUAGE_CODE = "es"
TIME_ZONE = "America/Bogota"
USE_I18N = True
USE_TZ = True
STATIC_URL = "static/"
DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"

LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "handlers": {"console": {"class": "logging.StreamHandler"}},
    "root": {"handlers": ["console"], "level": "INFO"},
}
```

### Qué realiza este paso

- excluye del repositorio archivos sensibles y temporales,
- define una plantilla de variables de entorno,
- crea la configuración principal del proyecto Django,
- enlaza la app `companies` con el proyecto.

### Explicación del código mostrado

- `.gitignore` evita versionar archivos sensibles o generados localmente,
- `.env.example` documenta las variables requeridas,
- `.env` contiene los valores reales del equipo,
- `settings.py` centraliza apps instaladas, middlewares, CORS, REST framework y logging.

### Aplicación de Twelve-Factor

- **Factor III. Config**: configuración separada del código.
- **Factor XI. Logs**: logs enviados a consola.

---

## 5. Crear la funcionalidad de autenticación

En este paso se implementa la validación de tokens de Firebase en el backend.

Crear el archivo `backend/companies/firebase_auth.py`:

```python
import os
import firebase_admin
from firebase_admin import credentials, auth
from rest_framework.authentication import BaseAuthentication
from rest_framework import exceptions
from dotenv import load_dotenv
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")

cred_path = os.getenv("FIREBASE_CREDENTIALS_PATH")

if not firebase_admin._apps:
    cred = credentials.Certificate(cred_path)
    firebase_admin.initialize_app(cred)

class FirebaseAuthentication(BaseAuthentication):
    def authenticate(self, request):
        auth_header = request.headers.get("Authorization")

        if not auth_header:
            return None

        if not auth_header.startswith("Bearer "):
            raise exceptions.AuthenticationFailed("Token mal formado")

        id_token = auth_header.split("Bearer ")[1]

        try:
            decoded_token = auth.verify_id_token(id_token)
            request.firebase_user = decoded_token
            return (None, None)
        except Exception:
            raise exceptions.AuthenticationFailed("Token inválido")
```

### Qué realiza este paso

- carga la credencial de Firebase,
- inicializa el SDK de administración,
- lee el token enviado en `Authorization`,
- valida si ese token es correcto,
- deja la información del usuario disponible en el request.

### Explicación del código mostrado

- `credentials.Certificate(...)` carga la credencial del backend,
- `firebase_admin.initialize_app(...)` inicializa Firebase,
- `BaseAuthentication` permite integrar autenticación personalizada en DRF,
- `auth.verify_id_token(...)` valida el token recibido.

### Aplicación de Twelve-Factor

- **Factor IV. Backing services**: Firebase se integra como servicio externo.

---

## 6. Crear la funcionalidad de acceso a datos

En este paso se implementa la conexión a MongoDB Atlas y se exponen las colecciones que utilizará el backend.

Crear el archivo `backend/companies/mongo.py`:

```python
import os
from pymongo import MongoClient
from dotenv import load_dotenv
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")

MONGODB_URI = os.getenv("MONGODB_URI")
MONGODB_DB_NAME = os.getenv("MONGODB_DB_NAME", "ecored_circular_db")

client = MongoClient(MONGODB_URI)
db = client[MONGODB_DB_NAME]

companies_collection = db["companies"]
material_listings_collection = db["material_listings"]
```

### Qué realiza este paso

- carga la configuración de MongoDB,
- crea el cliente de conexión,
- selecciona la base,
- deja disponibles dos colecciones para el resto del código.

### Explicación del código mostrado

- `MongoClient(MONGODB_URI)` crea el cliente de conexión,
- `db = client[MONGODB_DB_NAME]` selecciona la base,
- `db["companies"]` y `db["material_listings"]` referencian las colecciones que se usarán en el flujo del taller.

### Aplicación de Twelve-Factor

- **Factor IV. Backing services**: MongoDB Atlas se consume como servicio externo configurable.

---

## 7. Crear vistas y rutas

En este paso se construye la capa HTTP del backend. El objetivo es exponer endpoints mínimos para empresas, publicaciones y salud del servicio.

Crear el archivo `backend/companies/views.py`:

```python
from datetime import datetime, timezone
from bson import ObjectId
from rest_framework import viewsets, permissions, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from companies.mongo import companies_collection, material_listings_collection

class CompanyViewSet(viewsets.ViewSet):
    permission_classes = [permissions.IsAuthenticated]

    def list(self, request):
        uid = request.firebase_user.get("uid")
        companies = list(
            companies_collection.find(
                {"owner_uid": uid},
                {
                    "owner_uid": 1,
                    "name": 1,
                    "nit": 1,
                    "city": 1,
                    "sector": 1,
                    "created_at": 1,
                },
            )
        )

        for company in companies:
            company["id"] = str(company["_id"])
            del company["_id"]

        return Response(companies)

    def create(self, request):
        uid = request.firebase_user.get("uid")
        data = request.data

        document = {
            "owner_uid": uid,
            "name": data.get("name"),
            "nit": data.get("nit"),
            "city": data.get("city"),
            "sector": data.get("sector"),
            "created_at": datetime.now(timezone.utc).isoformat(),
        }

        result = companies_collection.insert_one(document)

        return Response({"id": str(result.inserted_id)}, status=status.HTTP_201_CREATED)


class MaterialListingViewSet(viewsets.ViewSet):
    permission_classes = [permissions.IsAuthenticated]

    def list(self, request):
        uid = request.firebase_user.get("uid")
        companies = list(companies_collection.find({"owner_uid": uid}, {"_id": 1}))
        company_ids = [company["_id"] for company in companies]

        items = list(material_listings_collection.find({"company_id": {"$in": company_ids}}))

        for item in items:
            item["id"] = str(item["_id"])
            item["company_id"] = str(item["company_id"])
            del item["_id"]

        return Response(items)

    def create(self, request):
        data = request.data

        document = {
            "company_id": ObjectId(data.get("company_id")),
            "material_type": data.get("material_type"),
            "quantity": float(data.get("quantity")),
            "unit": data.get("unit", "kg"),
            "location": data.get("location"),
            "status": data.get("status", "available"),
            "published_by": request.firebase_user.get("uid"),
            "created_at": datetime.now(timezone.utc).isoformat(),
        }

        result = material_listings_collection.insert_one(document)

        return Response({"id": str(result.inserted_id)}, status=status.HTTP_201_CREATED)


@api_view(["GET"])
@permission_classes([permissions.AllowAny])
def health(request):
    return Response({"status": "ok"})
```

Crear el archivo `backend/companies/urls.py`:

```python
from rest_framework.routers import DefaultRouter
from django.urls import path, include
from .views import CompanyViewSet, MaterialListingViewSet, health

router = DefaultRouter()
router.register(r"companies", CompanyViewSet, basename="companies")
router.register(r"material-listings", MaterialListingViewSet, basename="material-listings")

urlpatterns = [
    path("health/", health),
    path("", include(router.urls)),
]
```

Ajustar `backend/config/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("companies.urls")),
]
```

### Qué realiza este paso

- crea los endpoints de la API,
- conecta autenticación, acceso a datos y lógica de respuesta,
- expone la API del módulo de empresas.

### Explicación del código mostrado

- `CompanyViewSet` gestiona el registro y consulta de empresas,
- `MaterialListingViewSet` gestiona publicaciones asociadas a empresa,
- `health` permite verificar si el backend responde,
- `DefaultRouter` genera automáticamente las rutas REST de los viewsets.

### Aplicación de Twelve-Factor

- **Factor VI. Processes**: el backend expone un proceso web desacoplado.
- **Factor VII. Port binding**: el backend se publicará por puerto.
- **Factor XI. Logs**: la ejecución deja trazas visibles en consola.

---

## 8. Crear datos iniciales y validar ejecución

En este paso se construye un proceso administrativo independiente y se valida que el backend funcione correctamente.

Crear el archivo `backend/companies/management/commands/seed_demo.py`:

```python
from django.core.management.base import BaseCommand
from datetime import datetime, timezone
from companies.mongo import companies_collection, material_listings_collection

class Command(BaseCommand):
    help = "Crea datos demo en MongoDB"

    def handle(self, *args, **kwargs):
        existing = companies_collection.find_one({"nit": "900000001"})

        if not existing:
            result = companies_collection.insert_one({
                "owner_uid": "demo-owner",
                "name": "EcoRed Demo",
                "nit": "900000001",
                "city": "Bogotá",
                "sector": "Manufactura",
                "created_at": datetime.now(timezone.utc).isoformat(),
            })

            material_listings_collection.insert_one({
                "company_id": result.inserted_id,
                "material_type": "Cartón",
                "quantity": 80,
                "unit": "kg",
                "location": "Bogotá",
                "status": "available",
                "published_by": "demo-owner",
                "created_at": datetime.now(timezone.utc).isoformat(),
            })

        self.stdout.write(self.style.SUCCESS("Datos demo creados en MongoDB"))
```

Luego volver a la terminal integrada de VS Code y ejecutar:

### Revisar configuración

```powershell
python manage.py check
```

### Ejecutar proceso administrativo

```powershell
python manage.py seed_demo
```

### Ejecutar backend

```powershell
python manage.py runserver 8000
```

### Verificar health endpoint

Abrir:

```text
http://127.0.0.1:8000/api/health/
```

Respuesta esperada:

```json
{"status":"ok"}
```

### Qué realiza este paso

- crea un comando administrativo con datos de ejemplo,
- verifica que la configuración del proyecto sea válida,
- inicia el backend,
- comprueba que el servicio responde correctamente.

### Explicación del código mostrado

- `BaseCommand` permite crear comandos propios de Django,
- `handle()` es el punto de entrada del comando,
- primero se verifica si ya existe el dato demo,
- luego se insertan documentos de ejemplo,
- finalmente se imprime un mensaje de éxito.

### Aplicación de Twelve-Factor

- **Factor XII. Admin processes**: `seed_demo` como proceso administrativo aislado.
- **Factor VI. Processes**: proceso web y proceso administrativo diferenciados.
- **Factor IX. Disposability**: el backend puede reiniciarse y volver a responder.
- **Factor XI. Logs**: la ejecución queda visible en consola.
