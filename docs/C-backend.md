# C. Creación de backend

[← B. Configuración de backing services](B-backing-services.md) | [Volver al README](../README.md) | [Siguiente: D. Creación de frontend →](D-frontend.md)

## 1. Crear la estructura base del proyecto

```bash
mkdir ecored-circular
cd ecored-circular
mkdir backend frontend docs
git init
git checkout -b develop
```

### Crear `.gitignore` en la raíz

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

## 2. Crear entorno virtual e instalar dependencias

```bash
cd backend
python -m venv venv
```

### Linux/macOS

```bash
source venv/bin/activate
```

### Windows

```bash
venv\Scripts\activate
```

### Instalar dependencias

```bash
pip install django djangorestframework django-cors-headers python-dotenv firebase-admin pymongo gunicorn
pip freeze > requirements.txt
```

## 3. Crear proyecto Django

```bash
django-admin startproject config .
python manage.py startapp core
```

## 4. Configurar `settings.py`

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
    "core",
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
        "core.firebase_auth.FirebaseAuthentication",
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

## 5. Crear módulo de conexión a MongoDB

Archivo `backend/core/mongo.py`:

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

## 6. Definir estructura lógica de los documentos

### Colección `companies`

```json
{
  "_id": "ObjectId",
  "owner_uid": "firebase-user-id",
  "name": "EcoRed SAS",
  "nit": "900000001",
  "city": "Bogotá",
  "sector": "Manufactura",
  "created_at": "2026-04-08T12:00:00Z"
}
```

### Colección `material_listings`

```json
{
  "_id": "ObjectId",
  "company_id": "ObjectId",
  "material_type": "Cartón reciclable industrial",
  "quantity": 120.5,
  "unit": "kg",
  "location": "Bogotá",
  "status": "available",
  "published_by": "firebase-user-id",
  "created_at": "2026-04-08T12:30:00Z"
}
```

## 7. Crear proceso administrativo `seed_demo`

```bash
mkdir -p core/management/commands
```

Archivo `backend/core/management/commands/seed_demo.py`:

```python
from django.core.management.base import BaseCommand
from datetime import datetime, timezone
from core.mongo import companies_collection, material_listings_collection

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

## 8. Configurar Firebase en backend

Guardar en `backend/` el archivo:

```text
firebase-service-account.json
```

Archivo `backend/core/firebase_auth.py`:

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

## 9. Crear vistas y endpoints

Archivo `backend/core/views.py`:

```python
from datetime import datetime, timezone
from bson import ObjectId
from rest_framework import viewsets, permissions, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from core.mongo import companies_collection, material_listings_collection

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

Archivo `backend/core/urls.py`:

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

Editar `backend/config/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("core.urls")),
]
```

## Evidencia Twelve-Factor transversal en esta sección

- **Factor I. Codebase**: backend dentro de un solo repositorio.
- **Factor II. Dependencies**: `requirements.txt`.
- **Factor III. Config**: `.env`.
- **Factor IV. Backing services**: MongoDB Atlas y Firebase integrados como servicios externos.
- **Factor VI. Processes**: backend sin estado local persistente.
- **Factor XI. Logs**: salida por consola.
- **Factor XII. Admin processes**: `seed_demo`.
