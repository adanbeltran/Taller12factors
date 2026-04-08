# Taller 1 — Aplicación cloud-native mínima para EcoRed Circular con React, Django, Firebase y MongoDB Atlas, evidenciando los 12 factores

## 1. Propósito

Construir una aplicación mínima, pero completa en arquitectura, para evidenciar la aplicación rigurosa de los **12 factores** de una aplicación cloud-native.

Este taller se enfoca en diseñar, configurar y ejecutar una solución mínima con:

- **Frontend:** React
- **Backend:** Django + Django REST Framework
- **Autenticación:** Firebase Authentication
- **Persistencia:** MongoDB Atlas

El énfasis del taller **no está en evaluar amplitud funcional**, sino en demostrar que la solución fue construida siguiendo correctamente los principios de **Twelve-Factor App**.

---

## 2. Enfoque de evaluación del taller

Este taller **no evalúa la cantidad de funcionalidades implementadas**, sino la **correcta y rigurosa aplicación de los 12 factores** en una arquitectura mínima funcional.

Por tanto, el propósito no es construir una plataforma completa, sino una base pequeña, coherente y técnicamente bien justificada en términos de:

- codebase
- dependencias
- configuración
- backing services
- build, release y run
- procesos
- port binding
- concurrencia
- disposability
- paridad entre ambientes
- logs
- procesos administrativos

Durante la sesión, los estudiantes deben **seguir los pasos del taller**, ejecutar la aplicación mínima propuesta y al final **desarrollar o sustentar en sala** lo realizado, explicando cómo su solución evidencia cada uno de los factores.

---

## 3. Caso de estudio

### 3.1 Nombre del caso
**EcoRed Circular**

### 3.2 Descripción ampliada

**EcoRed Circular** es un caso de estudio orientado a la economía circular en Colombia. Su propósito es servir como base para una plataforma digital que articule actores vinculados al aprovechamiento, circulación y valorización de materiales reciclables o reutilizables.

El caso parte de una situación realista:

- muchas empresas generan residuos o subproductos con valor potencial,
- no siempre existe un mecanismo digital simple para registrar y publicar esos materiales,
- los actores que podrían reutilizarlos o transformarlos no siempre tienen visibilidad sobre la oferta disponible,
- la trazabilidad de materiales y la articulación entre oferta y demanda suele ser limitada.

Desde la perspectiva arquitectónica, este caso es adecuado para construir una aplicación cloud-native pequeña pero realista, apoyada en servicios externos administrados y diseñada desde el inicio con criterios de configuración externalizada, desacoplamiento y despliegue reproducible.

---

## 4. Módulos del caso de estudio

La solución completa de **EcoRed Circular** podría crecer hacia varios módulos funcionales.

### 4.1 Módulos posibles del sistema

- autenticación y acceso
- empresas
- publicaciones de materiales
- ciudadano
- reciclador
- transformador
- solicitudes o intención de compra
- sedes y georreferenciación
- administración
- impacto

### 4.2 Módulo que se desarrolla en este Taller 1

En este taller se desarrollará únicamente el **módulo de empresas** del caso de estudio.

Este módulo se implementa como una base mínima funcional que permite:

- autenticarse en la aplicación,
- registrar información básica de una empresa,
- asociar publicaciones de materiales a la empresa registrada,
- consultar la información registrada.

Aunque el flujo incluye autenticación y publicaciones de materiales, el enfoque del taller se organiza alrededor del **módulo de empresas** como núcleo funcional inicial de la arquitectura.

### 4.3 Proyección para iteraciones posteriores

Sobre esta base, en futuras iteraciones podrían desarrollarse otros módulos del caso, como:

- publicaciones avanzadas de materiales,
- solicitudes o intención de compra,
- perfiles de recicladores,
- perfiles de transformadores,
- sedes y georreferenciación,
- administración,
- indicadores de impacto.

---

## 5. Resultado esperado

Al finalizar, cada equipo debe dejar funcionando:

- un repositorio único con `backend/`, `frontend/` y `docs/`,
- backend Django REST conectado a MongoDB Atlas,
- frontend React autenticando con Firebase,
- registro de empresas,
- registro de publicaciones asociadas a empresa,
- consulta de información registrada,
- documentación explícita de evidencias Twelve-Factor.

El cierre del taller consiste en que los estudiantes **desarrollen o sustenten en sala** lo realizado, explicando cómo la solución construida evidencia de forma rigurosa los 12 factores.

---

## 6. Prerrequisitos

Se asume que cada equipo **ya tiene disponibles** los backing services requeridos:

- proyecto Firebase ya provisionado,
- autenticación ya habilitada en Firebase,
- proyecto y clúster de MongoDB Atlas ya provisionados,
- credenciales o datos de conexión ya disponibles,
- permisos de acceso previamente configurados.

Además, cada equipo debe contar con:

- Python 3.11 o superior
- Node.js LTS
- npm
- Git
- editor como VS Code

### Verificación rápida

```bash
python --version
node --version
npm --version
git --version
```

---

## 7. Evidencia esperada por factor

| Factor | Evidencia esperada |
|---|---|
| I. Codebase | Un solo repositorio con frontend y backend |
| II. Dependencies | `requirements.txt` y `package.json` |
| III. Config | `.env.example` y variables de entorno |
| IV. Backing services | MongoDB Atlas y Firebase conectados por configuración |
| V. Build, release, run | documento de release y ejecución separada |
| VI. Processes | proceso web y proceso administrativo diferenciados |
| VII. Port binding | backend por puerto y frontend por puerto |
| VIII. Concurrency | documento de escalado por procesos |
| IX. Disposability | reinicio limpio del backend |
| X. Dev/prod parity | tabla de ambientes |
| XI. Logs | logs visibles en consola/stdout |
| XII. Admin processes | comando `seed_demo` |

---

# Parte A. Preparación del proyecto

## Paso 1. Crear la estructura base

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

### Evidencia Twelve-Factor

**Factor I. Codebase**  
Un solo repositorio versionado con frontend, backend y documentación.

### Nota sobre versiones y ambientes

La entrega actual corresponde a la **versión 1.0** de la aplicación.

En futuras iteraciones podrán aparecer nuevas versiones, por ejemplo `v2.0` o `v2.1`, sin crear otro proyecto ni duplicar el repositorio. El proyecto sigue siendo el mismo (`ecored-circular`) y las nuevas versiones se gestionan dentro de la misma **codebase**.

Es importante diferenciar:

- **versión**: estado liberado del código, por ejemplo `v1.0` o `v2.0`
- **ambiente**: contexto donde esa versión se ejecuta, por ejemplo `dev`, `test` o `prod`

La misma versión puede pasar por varios ambientes. Por ejemplo:

- `v1.0` en `dev`
- `v1.0` en `test`
- `v1.0` en `prod`

Si mientras `v1.0` está en pruebas el equipo sigue desarrollando, esos cambios nuevos no deberían afectar lo que se está validando. Para eso, el ambiente de **test** debe trabajar sobre una **versión identificada y congelada**, mientras el desarrollo continúa sobre la misma codebase hacia una versión futura.

La idea corta es:
- v1.0 se prueba
- lo nuevo va a v1.1 o v2.0
- los fixes de v1.0 salen como v1.0.1

En consecuencia:

- existe una sola codebase,
- pueden existir varias versiones liberadas en el tiempo,
- una misma versión puede recorrer distintos ambientes,
- los ambientes no son proyectos distintos,
- la diferencia entre ambientes está principalmente en la configuración y los recursos externos asociados.

```


## Paso 2. Crear documento inicial de release

Crear `docs/release-notes.md`:

```md
# Release 0.1.0

## Build
- Backend Django REST
- Frontend React

## Release
- Variables MongoDB Atlas
- Variables Firebase

## Run
- Backend en puerto 8000
- Frontend en puerto 5173
```

### Evidencia Twelve-Factor

**Factor V. Build, release, run**  
La estructura del release se documenta desde el inicio.

---

# Parte B. Identificación de datos de conexión de los backing services

## Paso 3. Identificar datos de conexión de MongoDB Atlas

En este taller no se crea el proyecto, el clúster ni el usuario de base de datos.  
Se asume que esos recursos ya existen.

Lo que debe estar disponible es:

- `MONGODB_URI`
- `MONGODB_DB_NAME`

### Ejemplo

```env
MONGODB_URI=mongodb+srv://usuario:clave@cluster.xxxxx.mongodb.net/?retryWrites=true&w=majority
MONGODB_DB_NAME=ecored_circular_db
```

### Verificación mínima

Antes de continuar, confirmar que:

- la URI fue entregada correctamente,
- el usuario tiene permisos,
- el acceso de red ya fue habilitado,
- el nombre de base de datos está definido.

### Evidencia Twelve-Factor

**Factor III. Config**  
La conexión se entrega por variables de entorno.

**Factor IV. Backing services**  
MongoDB Atlas se usa como servicio externo.

---

## Paso 4. Identificar datos de conexión de Firebase

En este taller no se crea el proyecto Firebase ni se configura Authentication desde cero.  
Se asume que esos recursos ya existen.

Lo que debe estar disponible para el frontend es:

- `VITE_FIREBASE_API_KEY`
- `VITE_FIREBASE_AUTH_DOMAIN`
- `VITE_FIREBASE_PROJECT_ID`
- `VITE_FIREBASE_APP_ID`

Y para el backend:

- archivo `firebase-service-account.json`
- o ruta definida en `FIREBASE_CREDENTIALS_PATH`

### Ejemplo de variables frontend

```env
VITE_FIREBASE_API_KEY=xxxxx
VITE_FIREBASE_AUTH_DOMAIN=xxxxx.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=xxxxx
VITE_FIREBASE_APP_ID=xxxxx
```

### Ejemplo de variable backend

```env
FIREBASE_CREDENTIALS_PATH=./firebase-service-account.json
```

### Verificación mínima

Antes de continuar, confirmar que:

- el frontend tiene los datos del proyecto,
- el backend tiene acceso a la credencial de servicio,
- la autenticación ya fue habilitada previamente.

### Evidencia Twelve-Factor

**Factor III. Config**  
La configuración de Firebase vive fuera del código.

**Factor IV. Backing services**  
Firebase Authentication se integra como servicio externo.

---

# Parte C. Construcción del backend

## Paso 5. Crear entorno virtual e instalar dependencias

```bash
cd backend
python -m venv venv
```

### Activar entorno

#### Linux/macOS

```bash
source venv/bin/activate
```

#### Windows

```bash
venv\Scripts\activate
```

### Instalar dependencias

```bash
pip install django djangorestframework django-cors-headers python-dotenv firebase-admin pymongo gunicorn
pip freeze > requirements.txt
```

### Evidencia Twelve-Factor

**Factor II. Dependencies**  
Dependencias explícitas, declaradas y aisladas.

---

## Paso 6. Crear proyecto Django

```bash
django-admin startproject config .
python manage.py startapp core
```

---

## Paso 7. Crear `.env.example` y `.env`

Crear `backend/.env.example`:

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

Copiarlo a `backend/.env` y completar con los valores reales.

### Evidencia Twelve-Factor

**Factor III. Config**  
Toda la configuración sensible y cambiante está fuera del código.

---

## Paso 8. Configurar `settings.py`

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
    "handlers": {
        "console": {"class": "logging.StreamHandler"},
    },
    "root": {
        "handlers": ["console"],
        "level": "INFO",
    },
}
```

### Evidencia Twelve-Factor

- **Factor III. Config**
- **Factor XI. Logs**

---

## Paso 9. Crear módulo de conexión a MongoDB

Crear `backend/core/mongo.py`:

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

### Evidencia Twelve-Factor

**Factor IV. Backing services**  
MongoDB Atlas se consume como servicio externo configurable.

---

## Paso 10. Definir la estructura lógica de los documentos

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

### Relación conceptual

```text
companies (1) ---- (N) material_listings
```

---

## Paso 11. Crear proceso administrativo `seed_demo`

```bash
mkdir -p core/management/commands
```

Crear `backend/core/management/commands/seed_demo.py`:

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

Ejecutar:

```bash
python manage.py seed_demo
```

### Evidencia Twelve-Factor

**Factor XII. Admin processes**  
`seed_demo` se ejecuta como proceso administrativo aislado.

---

## Paso 12. Configurar Firebase en backend

Guardar en `backend/` el archivo:

```text
firebase-service-account.json
```

Crear `backend/core/firebase_auth.py`:

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

### Evidencia Twelve-Factor

- **Factor III. Config**
- **Factor IV. Backing services**
- **Factor VI. Processes**

---

## Paso 13. Crear vistas y endpoints

Crear `backend/core/views.py`:

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

Crear `backend/core/urls.py`:

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

---

## Paso 14. Probar backend

```bash
python manage.py runserver 8000
```

Abrir:

```text
http://127.0.0.1:8000/api/health/
```

Debe responder:

```json
{"status":"ok"}
```

### Evidencia Twelve-Factor

**Factor VII. Port binding**  
El backend expone su servicio por puerto.

---

# Parte D. Construcción del frontend

## Paso 15. Crear proyecto React

```bash
cd ../frontend
npm create vite@latest . -- --template react
npm install
npm install axios react-router-dom firebase
```

### Evidencia Twelve-Factor

**Factor II. Dependencies**

---

## Paso 16. Crear `.env.example`

Crear `frontend/.env.example`:

```env
VITE_API_URL=http://localhost:8000/api
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_APP_ID=
```

Copiarlo a `frontend/.env` y completar con los datos del proyecto ya existente.

### Evidencia Twelve-Factor

**Factor III. Config**

---

## Paso 17. Configurar Firebase en frontend

Crear `frontend/src/firebaseConfig.js`:

```javascript
import { initializeApp } from "firebase/app";
import { getAuth, signInWithPopup, GoogleAuthProvider } from "firebase/auth";

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);
export const provider = new GoogleAuthProvider();
export { signInWithPopup };
```

### Evidencia Twelve-Factor

**Factor IV. Backing services**

---

## Paso 18. Crear cliente HTTP

Crear `frontend/src/api.js`:

```javascript
import axios from "axios";

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("firebaseToken");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
```

---

## Paso 19. Crear páginas

### `frontend/src/LoginPage.jsx`

```javascript
import React from "react";
import { useNavigate } from "react-router-dom";
import { auth, provider, signInWithPopup } from "./firebaseConfig";

export default function LoginPage() {
  const navigate = useNavigate();

  const handleLogin = async () => {
    const result = await signInWithPopup(auth, provider);
    const token = await result.user.getIdToken();
    localStorage.setItem("firebaseToken", token);
    navigate("/company");
  };

  return (
    <div>
      <h2>Login</h2>
      <button onClick={handleLogin}>Ingresar con Google</button>
    </div>
  );
}
```

### `frontend/src/CompanyPage.jsx`

```javascript
import React, { useState } from "react";
import api from "./api";

export default function CompanyPage() {
  const [form, setForm] = useState({
    name: "",
    nit: "",
    city: "",
    sector: "",
  });
  const [msg, setMsg] = useState("");

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await api.post("/companies/", form);
      setMsg("Empresa creada");
    } catch {
      setMsg("Error al crear empresa");
    }
  };

  return (
    <div>
      <h2>Empresa</h2>
      {msg && <p>{msg}</p>}
      <form onSubmit={handleSubmit}>
        <input name="name" placeholder="Nombre" onChange={handleChange} />
        <input name="nit" placeholder="NIT" onChange={handleChange} />
        <input name="city" placeholder="Ciudad" onChange={handleChange} />
        <input name="sector" placeholder="Sector" onChange={handleChange} />
        <button type="submit">Guardar</button>
      </form>
    </div>
  );
}
```

### `frontend/src/MaterialsPage.jsx`

```javascript
import React, { useState, useEffect } from "react";
import api from "./api";

export default function MaterialsPage() {
  const [companies, setCompanies] = useState([]);
  const [items, setItems] = useState([]);
  const [form, setForm] = useState({
    company_id: "",
    material_type: "",
    quantity: "",
    unit: "kg",
    location: "",
    status: "available",
  });

  useEffect(() => {
    loadCompanies();
    loadItems();
  }, []);

  const loadCompanies = async () => {
    const res = await api.get("/companies/");
    setCompanies(res.data);
  };

  const loadItems = async () => {
    const res = await api.get("/material-listings/");
    setItems(res.data);
  };

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    await api.post("/material-listings/", form);
    loadItems();
  };

  return (
    <div>
      <h2>Materiales</h2>

      <form onSubmit={handleSubmit}>
        <select name="company_id" onChange={handleChange}>
          <option value="">Seleccione empresa</option>
          {companies.map((c) => (
            <option key={c.id} value={c.id}>{c.name}</option>
          ))}
        </select>

        <input name="material_type" placeholder="Tipo de material" onChange={handleChange} />
        <input name="quantity" placeholder="Cantidad" onChange={handleChange} />
        <input name="unit" placeholder="Unidad" onChange={handleChange} />
        <input name="location" placeholder="Ubicación" onChange={handleChange} />

        <button type="submit">Crear publicación</button>
      </form>

      <hr />

      {items.map((item) => (
        <div key={item.id}>
          <strong>{item.material_type}</strong> - {item.quantity} {item.unit} - {item.location}
        </div>
      ))}
    </div>
  );
}
```

---

## Paso 20. Configurar rutas

Editar `frontend/src/App.jsx`:

```javascript
import React from "react";
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";
import LoginPage from "./LoginPage";
import CompanyPage from "./CompanyPage";
import MaterialsPage from "./MaterialsPage";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/login">Login</Link> |{" "}
        <Link to="/company">Empresa</Link> |{" "}
        <Link to="/materials">Materiales</Link>
      </nav>

      <Routes>
        <Route path="/login" element={<LoginPage />} />
        <Route path="/company" element={<CompanyPage />} />
        <Route path="/materials" element={<MaterialsPage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

---

## Paso 21. Ejecutar frontend

```bash
npm run dev
```

Frontend en `http://localhost:5173`.

### Evidencia Twelve-Factor

**Factor VII. Port binding**  
El frontend expone su interfaz por puerto.

---

# Parte E. Logs, procesos y reinicio

## Paso 22. Validar logs

Con backend corriendo, realizar peticiones desde el frontend y observar la terminal del backend.

### Qué se evidencia

**Factor XI. Logs**  
Los logs salen por consola/stdout y se tratan como flujo de eventos.

---

## Paso 23. Validar procesos

### Proceso web

```bash
python manage.py runserver 8000
```

### Proceso administrativo

```bash
python manage.py seed_demo
```

Crear `docs/processes.md`:

```md
# Processes

## Proceso web
El proceso web atiende solicitudes HTTP y expone la API.

## Proceso administrativo
El proceso administrativo se ejecuta de forma separada al proceso web.
Ejemplo:
- seed_demo

## Conclusión
La persistencia está en MongoDB Atlas y la identidad en Firebase.
```

### Evidencia Twelve-Factor

**Factor VI. Processes**  
Se diferencia explícitamente entre proceso web y proceso administrativo.

---

## Paso 24. Validar reinicio del backend

1. detener backend con `Ctrl + C`,
2. reiniciar backend,
3. verificar que la aplicación sigue respondiendo,
4. confirmar que los datos persisten en MongoDB Atlas.

Crear `docs/disposability.md`:

```md
# Disposability

## Prueba realizada
1. iniciar backend
2. registrar empresa
3. registrar publicación
4. detener backend
5. reiniciar backend
6. consultar nuevamente

## Resultado
La aplicación reinicia rápidamente y los datos se conservan porque la persistencia está en MongoDB Atlas.
```

### Evidencia Twelve-Factor

**Factor IX. Disposability**

---

# Parte F. Evidencia final de los 12 factores

## Paso 25. Build, release y run

En `frontend/`:

```bash
npm run build
```

Completar `docs/release-notes.md`:

```md
# Release 0.1.0

## Build
- Frontend compilado con `npm run build`
- Backend preparado con `pip install -r requirements.txt`

## Release
- Variables MongoDB Atlas cargadas desde `.env`
- Variables Firebase cargadas desde `.env`
- APP_ENV=dev

## Run
- `python manage.py runserver 8000`
- `npm run dev`
```

### Evidencia Twelve-Factor

**Factor V. Build, release, run**

---

## Paso 26. Concurrency

Crear `docs/concurrency.md`:

```md
# Concurrency

El backend puede escalarse como múltiples procesos web del mismo tipo.

Ejemplo conceptual:
- web-1
- web-2
- web-3

Los procesos administrativos se ejecutan por separado:
- seed_demo
```

### Evidencia Twelve-Factor

**Factor VIII. Concurrency**

---

## Paso 27. Paridad entre ambientes

Crear `docs/environments.md`:

```md
# Environments

| Ambiente | Código | Base de datos | Auth | Config |
|---|---|---|---|---|
| dev | mismo repo | MongoDB Atlas dev | Firebase dev | .env.dev |
| test | mismo repo | MongoDB Atlas test | Firebase test | .env.test |
| staging | mismo repo | MongoDB Atlas staging | Firebase staging | variables del despliegue |
| prod | mismo repo | MongoDB Atlas prod | Firebase prod | variables del despliegue |
```

### Evidencia Twelve-Factor

**Factor X. Dev/prod parity**

---

## Paso 28. Checklist final

Crear `docs/checklist-12factor.md`:

```md
# Evidencia Twelve-Factor

| Factor | Evidencia |
|---|---|
| I. Codebase | Un solo repositorio con backend y frontend |
| II. Dependencies | requirements.txt y package.json |
| III. Config | .env.example en backend y frontend |
| IV. Backing services | MongoDB Atlas y Firebase configurados externamente |
| V. Build, release, run | release-notes.md y build del frontend |
| VI. Processes | processes.md; proceso web y administrativo diferenciados |
| VII. Port binding | backend 8000, frontend 5173 |
| VIII. Concurrency | concurrency.md |
| IX. Disposability | disposability.md |
| X. Dev/prod parity | environments.md |
| XI. Logs | logging a consola/stdout |
| XII. Admin processes | seed_demo |
```

---

# Parte G. Flujo de verificación

## Paso 29. Ejecutar flujo mínimo

Con backend y frontend corriendo:

1. entrar a `/login`,
2. iniciar sesión con Firebase,
3. ir a `/company` y registrar una empresa,
4. ir a `/materials` y registrar una publicación asociada,
5. verificar que la información aparece en el listado,
6. confirmar que existen documentos en:
   - `companies`
   - `material_listings`

---

# Parte H. Sustentación en sala

Al final del taller, los estudiantes deben **desarrollar o sustentar en sala** lo realizado.

La sustentación debe mostrar, con base en el repositorio y en la ejecución del sistema, cómo se evidencia cada uno de los 12 factores en la solución construida.

El foco de la sustentación no debe ponerse en la cantidad de pantallas ni en la complejidad funcional, sino en la **rigurosidad arquitectónica** de la implementación.

---

# Parte I. Entregables

Cada equipo debe dejar disponible:

- repositorio `ecored-circular`,
- backend funcionando,
- frontend funcionando,
- integración con MongoDB Atlas,
- integración con Firebase,
- colecciones `companies` y `material_listings` con datos,
- `docs/release-notes.md`,
- `docs/processes.md`,
- `docs/disposability.md`,
- `docs/concurrency.md`,
- `docs/environments.md`,
- `docs/checklist-12factor.md`.

---

# Parte J. Cierre del taller

Al terminar este Taller 1, el equipo habrá construido una base cloud-native mínima alrededor del **módulo de empresas** del caso de estudio EcoRed Circular.

La meta del taller no es evaluar cuántas funcionalidades tiene la aplicación, sino verificar que una solución mínima fue construida con una aplicación correcta, rigurosa y argumentable de los **12 factores**.

