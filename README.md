# Taller 1 — EcoRed Circular

Aplicación cloud-native mínima con **React**, **Django REST Framework**, **Firebase Authentication** y **MongoDB Atlas**, orientada a evidenciar rigurosamente los principios de **Twelve-Factor App**.

## Video de apoyo

[▶ Ver video en SharePoint](https://uniempresarial-my.sharepoint.com/:v:/g/personal/abeltran_uniempresarial_edu_co/IQBgLxQxE0NVSJV0fggITaXLAYKDKrG24zrSsFEI8c1u8tw?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D&e=Ay98Ym)

## Navegación del taller

### A. Caso de estudio
[Ir a la sección A](docs/A-caso-de-estudio.md)

### B. Configuración de backing services
[Ir a la sección B](docs/B-backing-services.md)

### C. Creación de backend
[Ir a la sección C](docs/C-backend.md)

### D. Creación de frontend
[Ir a la sección D](docs/D-frontend.md)

### E. Despliegue local
[Ir a la sección E](docs/E-despliegue-local.md)

---

## Alcance del taller

Este taller se concentra en una arquitectura mínima funcional alrededor del **módulo de empresas** del caso de estudio **EcoRed Circular**.

El énfasis del taller **no está en evaluar amplitud funcional**, sino en verificar la correcta aplicación de los **12 factores** a lo largo de todas las secciones:

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

## Resultado esperado

Al finalizar, cada equipo debe dejar funcionando:

- un repositorio único con `backend/`, `frontend/` y `docs/`,
- backend Django REST conectado a MongoDB Atlas,
- frontend React autenticando con Firebase,
- registro de empresas,
- registro de publicaciones asociadas a empresa,
- consulta de información registrada,
- documentación explícita de evidencias Twelve-Factor distribuida transversalmente en todas las secciones.

## Versión del taller

La entrega actual corresponde a la **versión 1.0**.

Esto significa que el proyecto se encuentra en su primer estado liberado y documentado. En futuras iteraciones podrán aparecer nuevas versiones, por ejemplo `v2.0` o `v2.1`, sin crear otro proyecto ni duplicar el repositorio.

Es importante diferenciar:

- **versión**: estado liberado del código, por ejemplo `v1.0` o `v2.0`
- **ambiente**: contexto donde esa versión se ejecuta, por ejemplo `dev`, `test` o `prod`

La misma versión puede pasar por varios ambientes. Si `v1.0` se encuentra en pruebas, el equipo puede continuar nuevos desarrollos hacia una versión futura, pero sin alterar automáticamente la versión congelada que está siendo validada.

## Estructura general del repositorio

```text
ecored-circular/
├── backend/
├── frontend/
├── docs/
│   ├── A-caso-de-estudio.md
│   ├── B-backing-services.md
│   ├── C-backend.md
│   ├── D-frontend.md
│   └── E-despliegue-local.md
└── README.md
```
