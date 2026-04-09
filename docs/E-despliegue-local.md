# E. Despliegue local

[← D. Creación de frontend](D-frontend.md) | [Volver al README](../README.md)

## 1. Probar backend

```bash
cd backend
python manage.py runserver 8000
```

Abrir:

```text
http://127.0.0.1:8000/api/health/
```

Respuesta esperada:

```json
{"status":"ok"}
```

## 2. Ejecutar frontend

```bash
cd ../frontend
npm run dev
```

Frontend en `http://localhost:5173`.

## 3. Flujo mínimo de verificación

Con backend y frontend corriendo:

1. entrar a `/login`,
2. iniciar sesión con Firebase,
3. ir a `/company` y registrar una empresa,
4. ir a `/materials` y registrar una publicación asociada,
5. verificar que la información aparece en el listado,
6. confirmar que existen documentos en:
   - `companies`
   - `material_listings`

## 4. Validar logs

Con backend corriendo, realizar peticiones desde el frontend y observar la terminal del backend.

## 5. Validar procesos

### Proceso web

```bash
python manage.py runserver 8000
```

### Proceso administrativo

```bash
python manage.py seed_demo
```

## 6. Validar reinicio del backend

1. detener backend con `Ctrl + C`,
2. reiniciar backend,
3. verificar que la aplicación sigue respondiendo,
4. confirmar que los datos persisten en MongoDB Atlas.

## 7. Build, release y run

En `frontend/`:

```bash
npm run build
```

## 8. Documentos de apoyo sugeridos

Dentro de `docs/` pueden mantenerse además:

- `release-notes.md`
- `processes.md`
- `disposability.md`
- `concurrency.md`
- `environments.md`
- `checklist-12factor.md`

## 9. Sustentación

La sustentacion sera a travez de un corto video (max 10 minutos) mostrando la funcionalidad y funcionamiento del codigo, es importante que se muestren como resolvieron los incovenientes presentados.

La sustentación debe mostrar, con base en el repositorio y en la ejecución del sistema, cómo se evidencia cada uno de los 12 factores en la solución construida.

## Evidencia Twelve-Factor transversal en esta sección

- **Factor V. Build, release, run**: separación entre construcción, release y ejecución.
- **Factor VI. Processes**: proceso web y proceso administrativo diferenciados.
- **Factor VII. Port binding**: backend en `8000`, frontend en `5173`.
- **Factor VIII. Concurrency**: el backend puede escalarse como múltiples procesos web del mismo tipo.
- **Factor IX. Disposability**: reinicio limpio del backend.
- **Factor X. Dev/prod parity**: misma aplicación con posibles despliegues `dev`, `test` y `prod`.
- **Factor XI. Logs**: salida a consola/stdout.
