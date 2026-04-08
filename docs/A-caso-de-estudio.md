# A. Caso de estudio

[← Volver al README](../README.md) | [Siguiente: B. Configuración de backing services →](B-backing-services.md)

## Propósito

Construir una aplicación mínima, pero completa en arquitectura, para evidenciar la aplicación rigurosa de los **12 factores** de una aplicación cloud-native.

La solución trabajará con:

- **Frontend:** React
- **Backend:** Django + Django REST Framework
- **Autenticación:** Firebase Authentication
- **Persistencia:** MongoDB Atlas

## Enfoque de evaluación

Este taller **no evalúa la cantidad de funcionalidades implementadas**, sino la **correcta y rigurosa aplicación de los 12 factores** en una arquitectura mínima funcional.

Durante la sesión, los estudiantes deben seguir los pasos, ejecutar la solución mínima propuesta y al final **desarrollar o sustentar en sala** lo realizado, explicando cómo su implementación evidencia los factores.

## Caso de estudio: EcoRed Circular

**EcoRed Circular** es un caso de estudio orientado a la economía circular en Colombia. Su propósito es servir como base para una plataforma digital que articule actores vinculados al aprovechamiento, circulación y valorización de materiales reciclables o reutilizables.

El caso parte de una situación realista:

- muchas empresas generan residuos o subproductos con valor potencial,
- no siempre existe un mecanismo digital simple para registrar y publicar esos materiales,
- los actores que podrían reutilizarlos o transformarlos no siempre tienen visibilidad sobre la oferta disponible,
- la trazabilidad de materiales y la articulación entre oferta y demanda suele ser limitada.

## Módulos posibles del sistema

La solución completa podría crecer hacia varios módulos:

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

## Módulo que se desarrolla en este Taller 1

En este taller se desarrollará únicamente el **módulo de empresas** del caso de estudio.

Este módulo se implementa como una base mínima funcional que permite:

- autenticarse en la aplicación,
- registrar información básica de una empresa,
- asociar publicaciones de materiales a la empresa registrada,
- consultar la información registrada.

Aunque el flujo incluye autenticación y publicaciones de materiales, el enfoque del taller se organiza alrededor del **módulo de empresas** como núcleo funcional inicial de la arquitectura.

## Proyección para iteraciones posteriores

Sobre esta base, en futuras iteraciones podrían desarrollarse otros módulos del caso, como:

- publicaciones avanzadas de materiales,
- solicitudes o intención de compra,
- perfiles de recicladores,
- perfiles de transformadores,
- sedes y georreferenciación,
- administración,
- indicadores de impacto.

## Prerrequisitos

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

## Evidencia Twelve-Factor transversal en esta sección

- **Factor I. Codebase**: una sola aplicación y un solo repositorio.
- **Factor X. Dev/prod parity**: una misma versión puede pasar por `dev`, `test` y `prod`.
- **Factor III. Config** y **Factor IV. Backing services**: la solución se apoya en servicios externos configurables.
