# C. Creación de backend

[← B. Configuración de backing services](B-backing-services.md) | [Volver al README](../README.md) | [Siguiente: D. Creación de frontend →](D-frontend.md)

## Propósito de esta sección

En esta sección se construirá el backend del proyecto usando **Visual Studio Code** como entorno principal de trabajo.

La lógica de trabajo será la siguiente:

1. crear desde la **terminal integrada de VS Code** toda la estructura base de carpetas y subcarpetas,
2. crear y editar los archivos desde el **explorador de VS Code**,
3. ejecutar y validar el backend desde la misma terminal integrada.

En coherencia con el alcance del taller, la app principal del backend será **`companies`**, ya que el foco funcional está en el **módulo de empresas** del caso de estudio.

---

## 1. Abrir VS Code y usar la terminal integrada

1. Abrir **Visual Studio Code**.
2. Abrir una carpeta de trabajo vacía o la ubicación donde se creará el proyecto.
3. Abrir la terminal integrada desde:
   - menú **Terminal > New Terminal**

A partir de este punto, todos los comandos del backend se ejecutarán desde esa terminal integrada.

---

## 2. Crear toda la estructura inicial del proyecto desde la terminal de VS Code

Ejecutar en la terminal integrada de VS Code:

```powershell
mkdir ecored-circular
cd ecored-circular
mkdir backend, frontend, docs
git init
git checkout -b develop
cd backend
mkdir companies
mkdir companies\management
mkdir companies\management\commands
```

Con esto queda creada la estructura base de carpetas y subcarpetas que se utilizará en el taller.

### Aclaración sobre versión, ambiente y flujo de trabajo con Git

Los comandos anteriores crean la estructura base del proyecto e inicializan el repositorio Git, pero no crean por sí mismos una versión liberada ni un ambiente de despliegue.

Es importante diferenciar:

- **proyecto**: `ecored-circular`, que corresponde a la única codebase del taller;
- **versión**: estado liberado del código, por ejemplo `v1.0`, `v1.0.1` o `v2.0`;
- **ambiente**: contexto donde una versión se ejecuta, por ejemplo `dev`, `test` o `prod`.

En este taller, los comandos Git iniciales solo preparan el control de versiones del proyecto y crean la rama de trabajo `develop`. La versión `v1.0` se definirá más adelante como release documentada del taller, y los ambientes posibles de despliegue no se representan como carpetas distintas dentro del repositorio.

En una situación real puede ocurrir lo siguiente al mismo tiempo:

- en **producción** se está ejecutando una versión estable, por ejemplo `v1.0`,
- en **pruebas** se está validando otra versión identificada, por ejemplo `v1.1`,
- en **desarrollo** el equipo continúa construyendo nuevas funcionalidades que formarán parte de una versión posterior, por ejemplo `v2.0`.

Esto no significa que existan proyectos distintos. Sigue existiendo una sola codebase, pero con diferentes **versiones liberadas** ejecutándose en distintos **ambientes** según su nivel de madurez.

La regla clave es esta:

- **producción** ejecuta la última versión aprobada,
- **pruebas** valida una versión congelada e identificada,
- **desarrollo** continúa con cambios nuevos que todavía no afectan la versión que se está probando ni la que ya está en producción.

De este modo, los nuevos desarrollos no alteran automáticamente la versión que se encuentra en validación ni la que ya está operando en producción.



### Estructura esperada en este punto

```text
ecored-circular/
├── backend/
│   └── companies/
│       └── management/
│           └── commands/
├── frontend/
├── docs/
```

### Evidencia Twelve-Factor transversal

- **Factor I. Codebase**: un solo repositorio y una sola estructura base del proyecto.
- **Factor XII. Admin processes**: desde el inicio queda prevista la carpeta para comandos administrativos.

---

## 3. Abrir la carpeta del proyecto en VS Code

Si aún no está abierta como carpeta principal, abrir en VS Code la carpeta:

```text
ecored-circular
```

A partir de este momento, los archivos se crearán y editarán desde el panel **Explorer** de VS Code.

---

## 4. Crear entorno virtual desde la terminal integrada de VS Code

Ubicados en `backend/`, ejecutar:

```powershell
python -m venv venv
```

### Activar entorno virtual

```powershell
venv\Scripts\activate
```

---

## 5. Instalar dependencias desde la terminal integrada de VS Code

Con el entorno virtual activado, ejecutar:

```powershell
pip install django djangorestframework django-cors-headers python-dotenv firebase-admin pymongo gunicorn
pip freeze > requirements.txt
```

### Evidencia Twelve-Factor transversal

- **Factor II. Dependencies**: dependencias declaradas explícitamente en `requirements.txt`.

---

## 6. Crear proyecto Django y la app `companies` desde la terminal integrada de VS Code

Todavía ubicados en `backend/`, ejecutar:

```powershell
django-admin startproject config .
python manage.py startapp companies
```

> Nota: si Django recrea la carpeta `companies`, verificar que dentro de ella exista la ruta `management\commands\`. Si no aparece, crearla nuevamente desde VS Code o desde la terminal.

---

## 7. Crear los archivos desde el explorador de VS Code

A partir de este punto, los archivos deben crearse desde el panel lateral de **VS Code**, usando **New File** sobre la carpeta correspondiente.

### Archivos que deben crearse o completarse

#### En la raíz del proyecto
- `.gitignore`

#### En `backend/`
- `.env.example`
- `.env`
- `requirements.txt` si no quedó generado
- `firebase-service-account.json`

#### En `backend\config\`
- `settings.py`
- `urls.py`

#### En `backend\companies\`
- `mongo.py`
- `firebase_auth.py`
- `views.py`
- `urls.py`

#### En `backend\companies\management\commands\`
- `seed_demo.py`

---

## 8. Crear `.gitignore` desde VS Code

Crear el archivo `.gitignore` en la raíz del proyecto con este contenido:

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

### Evidencia Twelve-Factor transversal

- **Factor III. Config**: la configuración sensible no entra al repositorio.

---

## 9. Crear `backend/.env.example` y `backend/.env` desde VS Code

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

### Evidencia Twelve-Factor transversal

- **Factor III. Config**: configuración separada del código.
- **Factor IV. Backing services**: conexión a MongoDB Atlas y Firebase mediante variables de entorno.

---

## 10. Editar `backend/config/settings.py` desde VS Code

Abrir `backend/config/settings.py` y reemplazar su contenido esencial por:

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

---

## 11. Crear `backend/companies/mongo.py` desde VS Code

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

---

## 12. Crear `backend/companies/firebase_auth.py` desde VS Code

Guardar en `backend/` el archivo `firebase-service-account.json` y luego crear `backend/companies/firebase_auth.py`:

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

---

## 13. Crear `backend/companies/views.py` desde VS Code

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
                {"owner_uid": 1, "name": 1, "nit": 1, "city": 1, "sector": 1, "created_at": 1}
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
            "created_at": datetime.now(timezone.utc).isoformat()
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
            "created_at": datetime.now(timezone.utc).isoformat()
        }
        result = material_listings_collection.insert_one(document)
        return Response({"id": str(result.inserted_id)}, status=status.HTTP_201_CREATED)

@api_view(["GET"])
@permission_classes([permissions.AllowAny])
def health(request):
    return Response({"status": "ok"})
```

---

## 14. Crear `backend/companies/urls.py` y ajustar `backend/config/urls.py` desde VS Code

### `backend/companies/urls.py`

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

### `backend/config/urls.py`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("companies.urls")),
]
```

---

## 15. Crear `backend/companies/management/commands/seed_demo.py` desde VS Code

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
                "created_at": datetime.now(timezone.utc).isoformat()
            })

            material_listings_collection.insert_one({
                "company_id": result.inserted_id,
                "material_type": "Cartón",
                "quantity": 80,
                "unit": "kg",
                "location": "Bogotá",
                "status": "available",
                "published_by": "demo-owner",
                "created_at": datetime.now(timezone.utc).isoformat()
            })

        self.stdout.write(self.style.SUCCESS("Datos demo creados en MongoDB"))
```

---

## 16. Validar el backend desde la terminal integrada de VS Code

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

---

## Evidencia Twelve-Factor transversal en esta sección

- **Factor I. Codebase**: backend dentro de un solo repositorio.
- **Factor II. Dependencies**: dependencias declaradas en `requirements.txt`.
- **Factor III. Config**: `.env` y `.env.example`.
- **Factor IV. Backing services**: integración con MongoDB Atlas y Firebase.
- **Factor VI. Processes**: proceso web y proceso administrativo separados.
- **Factor XI. Logs**: salida por consola en la terminal de VS Code.
- **Factor XII. Admin processes**: `seed_demo` como proceso aislado.
