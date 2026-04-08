# B. Configuración de backing services

[← A. Caso de estudio](A-caso-de-estudio.md) | [Volver al README](../README.md) | [Siguiente: C. Creación de backend →](C-backend.md)

## Objetivo de la sección

Identificar y registrar los datos de conexión de los servicios externos que la aplicación consumirá como **backing services**.

En este taller no se crea el proyecto Firebase ni el proyecto de MongoDB Atlas. Se asume que ambos ya existen y que el estudiante cuenta con acceso a ellos.

## 1. Datos de conexión de MongoDB Atlas

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

## 2. Datos de conexión de Firebase

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

## 3. Variables de entorno esperadas

### Backend — `.env.example`

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

### Frontend — `.env.example`

```env
VITE_API_URL=http://localhost:8000/api
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_APP_ID=
```

## Evidencia Twelve-Factor transversal en esta sección

- **Factor III. Config**: la configuración vive fuera del código.
- **Factor IV. Backing services**: MongoDB Atlas y Firebase se consumen como servicios externos.
- **Factor X. Dev/prod parity**: los ambientes pueden cambiar por configuración sin duplicar la aplicación.
