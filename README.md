# Taller 1 — Aplicación cloud-native mínima con React, Django, Firebase y Oracle OCI, evidenciando los 12 factores

## 1. Propósito

Construir una aplicación mínima, pero completa en arquitectura, para evidenciar los **12 factores** de una aplicación cloud-native.

La metodología **Twelve-Factor App** propone, entre otros principios:

- un solo código base,
- dependencias explícitas,
- configuración en variables de entorno,
- servicios externos conectables,
- separación entre build, release y run,
- procesos sin estado,
- publicación por puertos,
- escalado por procesos,
- reinicio robusto,
- paridad entre ambientes,
- logs por `stdout`,
- procesos administrativos one-off.

Referencia oficial:

- https://12factor.net/es/

---

## 2. Caso de estudio

Se usará una versión mínima de **EcoRed Circular**.

En este taller, el flujo será:

1. iniciar sesión con Firebase,
2. registrar una empresa,
3. publicar un material reciclable,
4. consultar sus publicaciones.

### Alcance intencionalmente reducido

No se implementan en esta versión:

- sedes,
- Mapbox,
- i18n completo,
- múltiples roles,
- logística,
- marketplace expandido.

Esto permite concentrar el tiempo en arquitectura cloud-native + evidencia rigurosa de los 12 factores.

---

## 3. Resultado esperado

Al finalizar, cada equipo debe dejar funcionando:

- un repositorio único con `backend/`, `frontend/` y `docs/`,
- una **Oracle Autonomous Database** creada en OCI,
- un usuario de base de datos para la aplicación,
- backend Django REST conectado a Oracle,
- frontend React con login usando Firebase,
- formulario para empresa,
- formulario para publicación,
- listado de publicaciones,
- evidencia documentada de los 12 factores.

---

## 4. Qué se implementa y qué no

### Sí se implementa

- React
- Django REST Framework
- Firebase Authentication
- Oracle Autonomous Database
- variables de entorno
- endpoint de salud
- módulo `Company`
- módulo `MaterialListing`
- documentación de evidencia Twelve-Factor

### No se implementa en esta versión

- Mapbox operativo
- i18n completo
- sedes
- múltiples perfiles de actor
- panel administrativo
- marketplace ampliado

---

## 5. Prerrequisitos

Cada equipo debe tener:

- Python 3.11 o superior
- Node.js LTS
- Git
- cuenta en Firebase
- acceso a OCI con permisos para crear Autonomous Database
- editor como VS Code

### Verificación rápida

```bash
python --version
node --version
npm --version
git --version
```

---

## 6. Evidencia esperada por factor

| Factor | Evidencia esperada |
|---|---|
| I. Codebase | Un solo repositorio con frontend y backend |
| II. Dependencies | `requirements.txt` y `package.json` |
| III. Config | `.env.example` y variables de entorno |
| IV. Backing services | Oracle y Firebase conectados por configuración |
| V. Build, release, run | documento de release y ejecución separada |
| VI. Processes | backend sin estado local persistente |
| VII. Port binding | backend en 8000 y frontend en 3000 |
| VIII. Concurrency | documento de escalado por procesos |
| IX. Disposability | reinicio limpio del backend |
| X. Dev/prod parity | tabla de ambientes |
| XI. Logs | logs visibles en consola |
| XII. Admin processes | `migrate` y `seed_demo` |

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
frontend/.env
__pycache__/
*.pyc
node_modules/
dist/
build/
```

### Evidencia Twelve-Factor

**Factor I. Codebase**  
Un solo repositorio versionado.

---

## Paso 2. Crear documento inicial de release

Crear `docs/release-notes.md`:

```md
# Release 0.1.0

## Build
- Backend Django REST
- Frontend React

## Release
- Variables Oracle
- Variables Firebase

## Run
- Backend en puerto 8000
- Frontend en puerto 3000
```

### Evidencia Twelve-Factor

**Factor V. Build, release, run**  
Este archivo se completará al final.

---

# Parte B. Crear la base de datos en Oracle OCI

## Paso 3. Aprovisionar Oracle Autonomous Database

En OCI:

1. Ir a **Oracle Database → Autonomous Database**.
2. Elegir compartimento y región.
3. Clic en **Create Autonomous Database**.
4. Completar:
   - **Display name**: `ecored-circular-db`
   - **Database name**: `ECOREDDB`
   - **Workload type**: `Transaction Processing`
   - tamaño mínimo permitido
   - contraseña de `ADMIN`
5. Crear la instancia.
6. Esperar a que el estado cambie a **Available**.

### Evidencia Twelve-Factor

**Factor IV. Backing services**  
La base de datos es un servicio externo, no parte del código.

### Evidencia sugerida

Guardar en `docs/` una captura o nota con:

- nombre de la instancia,
- región,
- compartimento,
- estado `Available`.

---

## Paso 4. Obtener datos de conexión

Entrar a la instancia → **Database connection**.

Registrar estos datos:

- `ORACLE_DB_HOST`
- `ORACLE_DB_PORT`
- `ORACLE_DB_SERVICE`

### Nota

Si el curso usa conexión **TLS sin wallet**, usar el connection string correspondiente según la configuración de la instancia.

---

## Paso 5. Crear usuario de aplicación

Conectarse como `ADMIN` y ejecutar:

```sql
CREATE USER ecored_app IDENTIFIED BY "ClaveSegura_2026";
GRANT CREATE SESSION TO ecored_app;
GRANT CREATE TABLE TO ecored_app;
GRANT CREATE SEQUENCE TO ecored_app;
GRANT CREATE VIEW TO ecored_app;
GRANT CREATE TRIGGER TO ecored_app;
GRANT UNLIMITED TABLESPACE TO ecored_app;
```

### Recomendación

No usar `ADMIN` como usuario de la aplicación.  
La aplicación debe conectarse con `ecored_app`.

---

## Paso 6. Probar conectividad

Antes de tocar Django, confirmar:

- instancia en `Available`,
- usuario `ecored_app` creado,
- host, puerto y service name correctos.

Guardar evidencia en `docs/`.

---

# Parte C. Construcción del backend

## Paso 7. Crear entorno virtual e instalar dependencias

```bash
cd backend
python -m venv venv
```

### Activar entorno

#### Windows

```bash
venv\Scripts\activate
```

#### Linux/macOS

```bash
source venv/bin/activate
```

### Instalar dependencias

```bash
pip install django djangorestframework django-cors-headers python-dotenv firebase-admin oracledb gunicorn
pip freeze > requirements.txt
```

### Evidencia Twelve-Factor

**Factor II. Dependencies**  
Dependencias explícitas y aisladas.

---

## Paso 8. Crear proyecto Django

```bash
django-admin startproject config .
python manage.py startapp core
```

Verificar que existen:

- `manage.py`
- `config/settings.py`
- `core/models.py`

---

## Paso 9. Crear `.env.example` y `.env`

Crear `backend/.env.example`:

```env
DJANGO_SECRET_KEY=
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
APP_ENV=dev
PORT=8000

ORACLE_DB_USER=ecored_app
ORACLE_DB_PASSWORD=
ORACLE_DB_HOST=
ORACLE_DB_PORT=1522
ORACLE_DB_SERVICE=

CORS_ALLOWED_ORIGINS=http://localhost:3000

FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=
```

Copiarlo a `backend/.env` y completar valores reales.

### Evidencia Twelve-Factor

**Factor III. Config**  
Toda la configuración sensible y cambiante está fuera del código.

---

## Paso 10. Configurar `settings.py`

Reemplazar lo esencial de `backend/config/settings.py` por:

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

TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.oracle",
        "NAME": os.getenv("ORACLE_DB_SERVICE"),
        "USER": os.getenv("ORACLE_DB_USER"),
        "PASSWORD": os.getenv("ORACLE_DB_PASSWORD"),
        "HOST": os.getenv("ORACLE_DB_HOST"),
        "PORT": os.getenv("ORACLE_DB_PORT", "1522"),
    }
}

CORS_ALLOWED_ORIGINS = os.getenv("CORS_ALLOWED_ORIGINS", "http://localhost:3000").split(",")

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

### Verificar configuración

```bash
python manage.py check
```

### Evidencia Twelve-Factor

- **Factor III. Config**
- **Factor IV. Backing services**
- **Factor XI. Logs**

---

# Parte D. Crear tablas del taller

## Paso 11. Definir modelos

Editar `backend/core/models.py`:

```python
from django.db import models

class Company(models.Model):
    owner_uid = models.CharField(max_length=128)
    name = models.CharField(max_length=150)
    nit = models.CharField(max_length=50, unique=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name


class MaterialListing(models.Model):
    STATUS_CHOICES = [
        ("available", "Available"),
        ("reserved", "Reserved"),
        ("closed", "Closed"),
    ]

    company = models.ForeignKey(Company, on_delete=models.CASCADE, related_name="listings")
    title = models.CharField(max_length=150)
    category = models.CharField(max_length=100)
    quantity = models.DecimalField(max_digits=10, decimal_places=2)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default="available")
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

## Paso 12. Crear tablas con migraciones

```bash
python manage.py makemigrations
python manage.py migrate
```

### Tablas de negocio creadas

- `core_company`
- `core_materiallisting`

### Tablas internas de Django

Entre otras:

- `django_migrations`
- `django_content_type`
- `auth_permission`
- `auth_group`
- `auth_group_permissions`
- `auth_user`
- `auth_user_groups`
- `auth_user_user_permissions`
- `django_admin_log`
- `django_session`

### Relación entre tablas de negocio

```text
core_company (1) ---- (N) core_materiallisting
```

### Evidencia Twelve-Factor

**Factor XII. Admin processes**  
`makemigrations` y `migrate` son procesos administrativos puntuales.

---

## Paso 13. Crear seed de ejemplo

```bash
mkdir -p core/management/commands
```

Crear `core/management/commands/seed_demo.py`:

```python
from django.core.management.base import BaseCommand
from core.models import Company

class Command(BaseCommand):
    help = "Crea datos demo"

    def handle(self, *args, **kwargs):
        Company.objects.get_or_create(
            nit="900000001",
            defaults={
                "owner_uid": "demo-owner",
                "name": "EcoRed Demo"
            }
        )
        self.stdout.write(self.style.SUCCESS("Datos demo creados"))
```

Ejecutar:

```bash
python manage.py seed_demo
```

### Evidencia Twelve-Factor

**Factor XII. Admin processes**  
El seed se ejecuta como proceso one-off.

---

# Parte E. API REST y autenticación

## Paso 14. Crear serializers

Crear `backend/core/serializers.py`:

```python
from rest_framework import serializers
from .models import Company, MaterialListing

class CompanySerializer(serializers.ModelSerializer):
    class Meta:
        model = Company
        fields = "__all__"
        read_only_fields = ["owner_uid", "created_at"]


class MaterialListingSerializer(serializers.ModelSerializer):
    class Meta:
        model = MaterialListing
        fields = "__all__"
        read_only_fields = ["created_at"]
```

---

## Paso 15. Configurar Firebase en backend

Crear `backend/core/firebase_auth.py`:

```python
import os
import firebase_admin
from firebase_admin import credentials, auth
from rest_framework.authentication import BaseAuthentication
from rest_framework import exceptions

if not firebase_admin._apps:
    cred_dict = {
        "type": "service_account",
        "project_id": os.getenv("FIREBASE_PROJECT_ID"),
        "client_email": os.getenv("FIREBASE_CLIENT_EMAIL"),
        "private_key": os.getenv("FIREBASE_PRIVATE_KEY", "").replace("\n", "\n"),
        "token_uri": "https://oauth2.googleapis.com/token",
    }
    cred = credentials.Certificate(cred_dict)
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

> Nota: si al copiar necesitas convertir saltos de línea reales en la llave privada, usa:
> `os.getenv("FIREBASE_PRIVATE_KEY", "").replace("\\n", "\n")`

---

## Paso 16. Crear vistas

Editar `backend/core/views.py`:

```python
from rest_framework import viewsets, permissions
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response

from .models import Company, MaterialListing
from .serializers import CompanySerializer, MaterialListingSerializer

class CompanyViewSet(viewsets.ModelViewSet):
    serializer_class = CompanySerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        uid = self.request.firebase_user.get("uid")
        return Company.objects.filter(owner_uid=uid)

    def perform_create(self, serializer):
        uid = self.request.firebase_user.get("uid")
        serializer.save(owner_uid=uid)


class MaterialListingViewSet(viewsets.ModelViewSet):
    serializer_class = MaterialListingSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        uid = self.request.firebase_user.get("uid")
        return MaterialListing.objects.filter(company__owner_uid=uid)


@api_view(["GET"])
@permission_classes([permissions.AllowAny])
def health(request):
    return Response({"status": "ok"})
```

### Evidencia Twelve-Factor

**Factor VI. Processes**  
El backend no guarda estado de negocio local; persistencia en Oracle e identidad en Firebase.

---

## Paso 17. Crear rutas

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

## Paso 18. Probar backend

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
La aplicación expone su servicio por puerto.

---

# Parte F. Construcción del frontend

## Paso 19. Crear proyecto React

En otra terminal:

```bash
cd ../frontend
npx create-react-app .
npm install axios react-router-dom firebase
```

### Evidencia Twelve-Factor

**Factor II. Dependencies**

---

## Paso 20. Crear `.env.example`

Crear `frontend/.env.example`:

```env
REACT_APP_API_URL=http://localhost:8000/api
REACT_APP_FIREBASE_API_KEY=
REACT_APP_FIREBASE_AUTH_DOMAIN=
REACT_APP_FIREBASE_PROJECT_ID=
REACT_APP_FIREBASE_APP_ID=
```

Copiarlo a `frontend/.env` y completar.

### Evidencia Twelve-Factor

**Factor III. Config**

---

## Paso 21. Configurar Firebase

Crear `frontend/src/firebaseConfig.js`:

```javascript
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";

const firebaseConfig = {
  apiKey: process.env.REACT_APP_FIREBASE_API_KEY,
  authDomain: process.env.REACT_APP_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.REACT_APP_FIREBASE_PROJECT_ID,
  appId: process.env.REACT_APP_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

### En Firebase

- crear proyecto,
- habilitar Authentication,
- activar Email/Password,
- crear usuario de prueba.

### Evidencia Twelve-Factor

**Factor IV. Backing services**

---

## Paso 22. Crear cliente HTTP

Crear `frontend/src/api.js`:

```javascript
import axios from "axios";

const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
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

## Paso 23. Crear páginas

### `frontend/src/LoginPage.js`

```javascript
import React, { useState } from "react";
import { signInWithEmailAndPassword } from "firebase/auth";
import { auth } from "./firebaseConfig";
import { useNavigate } from "react-router-dom";

export default function LoginPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [msg, setMsg] = useState("");
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const cred = await signInWithEmailAndPassword(auth, email, password);
      const token = await cred.user.getIdToken();
      localStorage.setItem("firebaseToken", token);
      navigate("/company");
    } catch {
      setMsg("Error de autenticación");
    }
  };

  return (
    <div>
      <h2>Login</h2>
      {msg && <p>{msg}</p>}
      <form onSubmit={handleSubmit}>
        <input type="email" placeholder="Correo" value={email} onChange={(e) => setEmail(e.target.value)} />
        <input type="password" placeholder="Contraseña" value={password} onChange={(e) => setPassword(e.target.value)} />
        <button type="submit">Ingresar</button>
      </form>
    </div>
  );
}
```

### `frontend/src/CompanyPage.js`

```javascript
import React, { useState } from "react";
import api from "./api";

export default function CompanyPage() {
  const [form, setForm] = useState({ name: "", nit: "" });
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
        <button type="submit">Guardar</button>
      </form>
    </div>
  );
}
```

### `frontend/src/MaterialsPage.js`

```javascript
import React, { useState, useEffect } from "react";
import api from "./api";

export default function MaterialsPage() {
  const [companies, setCompanies] = useState([]);
  const [items, setItems] = useState([]);
  const [form, setForm] = useState({
    company: "",
    title: "",
    category: "",
    quantity: "",
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
        <select name="company" onChange={handleChange}>
          <option value="">Seleccione empresa</option>
          {companies.map((c) => (
            <option key={c.id} value={c.id}>{c.name}</option>
          ))}
        </select>

        <input name="title" placeholder="Título" onChange={handleChange} />
        <input name="category" placeholder="Categoría" onChange={handleChange} />
        <input name="quantity" placeholder="Cantidad" onChange={handleChange} />
        <select name="status" onChange={handleChange}>
          <option value="available">Disponible</option>
          <option value="reserved">Reservado</option>
          <option value="closed">Cerrado</option>
        </select>

        <button type="submit">Crear publicación</button>
      </form>

      <hr />

      {items.map((item) => (
        <div key={item.id}>
          <strong>{item.title}</strong> - {item.category} - {item.quantity}
        </div>
      ))}
    </div>
  );
}
```

---

## Paso 24. Configurar rutas

Editar `frontend/src/App.js`:

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

## Paso 25. Ejecutar frontend

```bash
npm start
```

Frontend en `http://localhost:3000`.

### Evidencia Twelve-Factor

**Factor VII. Port binding**

---

# Parte G. Logs, procesos y reinicio

## Paso 26. Validar logs

Con backend corriendo, hacer peticiones desde el frontend y observar la terminal del backend.

### Qué se evidencia

**Factor XI. Logs**  
Los logs salen por consola, como flujo de eventos, no a archivos locales.

### Evidencia sugerida

Guardar una captura de la terminal del backend.

---

## Paso 27. Validar procesos

Diferenciar claramente:

### Proceso web

```bash
python manage.py runserver 8000
```

### Procesos administrativos

```bash
python manage.py migrate
python manage.py seed_demo
```

Crear `docs/processes.md`:

```md
# Processes

La aplicación se ejecuta como procesos sin estado.
El proceso web atiende peticiones HTTP.
Los procesos administrativos se ejecutan por separado:
- migrate
- seed_demo

La persistencia está en Oracle y la identidad en Firebase.
```

### Qué se evidencia

**Factor VI. Processes**  
El backend es stateless; persistencia en Oracle e identidad en Firebase.

---

## Paso 28. Validar reinicio

1. Detener backend con `Ctrl + C`.
2. Reiniciar:

```bash
python manage.py runserver 8000
```

3. Verificar que la app vuelve a responder sin reparación manual.

Crear `docs/disposability.md`:

```md
# Disposability

La aplicación reinicia rápidamente.
No guarda estado persistente en memoria local.
La persistencia está en Oracle y la identidad en Firebase.
```

### Qué se evidencia

**Factor IX. Disposability**  
Inicio rápido y apagado seguro.

---

# Parte H. Evidencia final de los 12 factores

## Paso 29. Build, release y run

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
- Variables Oracle cargadas desde `.env`
- Variables Firebase cargadas desde `.env`
- APP_ENV=dev

## Run
- `python manage.py runserver 8000`
- `npm start`
```

### Qué se evidencia

**Factor V. Build, release, run**  
Separación entre construcción, release y ejecución.

---

## Paso 30. Concurrency

Crear `docs/concurrency.md`:

```md
# Concurrency

El backend puede escalarse como múltiples procesos web del mismo tipo.
Ejemplo conceptual:
- web-1
- web-2
- web-3

Los procesos administrativos se ejecutan por separado:
- migrate
- seed_demo
```

### Qué se evidencia

**Factor VIII. Concurrency**  
El escalado se plantea por procesos.

---

## Paso 31. Paridad entre ambientes

Crear `docs/environments.md`:

```md
# Ambientes

| Ambiente | Código | Base de datos | Auth | Config |
|---|---|---|---|---|
| dev | mismo repo | Oracle | Firebase | .env.dev |
| test | mismo repo | Oracle | Firebase | .env.test |
| staging | mismo repo | Oracle | Firebase | variables del despliegue |
| prod | mismo repo | Oracle | Firebase | variables del despliegue |
```

### Qué se evidencia

**Factor X. Dev/prod parity**  
Mismo código y mismo tipo de servicios; cambia la configuración.

---

## Paso 32. Checklist final

Crear `docs/checklist-12factor.md`:

```md
# Evidencia Twelve-Factor

| Factor | Evidencia |
|---|---|
| I. Codebase | Un solo repositorio con backend y frontend |
| II. Dependencies | requirements.txt y package.json |
| III. Config | .env.example en backend y frontend |
| IV. Backing services | Oracle y Firebase configurados externamente |
| V. Build, release, run | release-notes.md y build del frontend |
| VI. Processes | processes.md; backend sin estado local persistente |
| VII. Port binding | backend 8000, frontend 3000 |
| VIII. Concurrency | concurrency.md |
| IX. Disposability | disposability.md |
| X. Dev/prod parity | environments.md |
| XI. Logs | logging a consola/stdout |
| XII. Admin processes | migrate y seed_demo |
```

---

# Parte I. Flujo completo de prueba

## Paso 33. Ejecutar flujo funcional

Con backend y frontend corriendo:

1. entrar a `/login`,
2. iniciar sesión con usuario de Firebase,
3. ir a `/company` y crear una empresa,
4. ir a `/materials` y crear una publicación,
5. verificar que la publicación aparece en el listado,
6. confirmar en Oracle que existen registros en:
   - `core_company`
   - `core_materiallisting`

---

# Parte J. Resumen técnico de base de datos

## Tablas de negocio creadas por el taller

- `core_company`
- `core_materiallisting`

## Relación

```text
core_company (1) ---- (N) core_materiallisting
```

## Propósito

- `core_company`: almacena empresas registradas por usuarios autenticados.
- `core_materiallisting`: almacena publicaciones de materiales reciclables asociadas a una empresa.

## Tablas internas de soporte de Django

Entre otras:

- `django_migrations`
- `django_content_type`
- `auth_permission`
- `auth_group`
- `auth_group_permissions`
- `auth_user`
- `auth_user_groups`
- `auth_user_user_permissions`
- `django_admin_log`
- `django_session`

---

# Parte K. Entregables

Cada equipo debe entregar:

- repositorio `ecored-circular`
- Oracle Autonomous Database creada
- usuario `ecored_app`
- tablas `core_company` y `core_materiallisting` creadas y con datos
- backend funcionando
- frontend funcionando
- `docs/release-notes.md`
- `docs/processes.md`
- `docs/disposability.md`
- `docs/concurrency.md`
- `docs/environments.md`
- `docs/checklist-12factor.md`

---

# Parte L. Qué queda como ampliación

Sobre esta base pueden extender después:

- sedes
- Mapbox
- internacionalización
- perfiles ciudadano/reciclador/transformador
- solicitudes de compra
- despliegue completo en OCI con VM y contenedores

