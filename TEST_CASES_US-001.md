# SUITE DE CASOS DE PRUEBA: SEGURIDAD Y OPTIMIZACIÓN DE INFRAESTRUCTURA (SOFKIANOS-MVP)

Este documento detalla los casos de prueba diseñados para validar la arquitectura, seguridad y flujos de CI/CD del ecosistema **SofkianOS-MVP**.

---

##  TC_001: Validación de Arquitectura Multi-stage en Componentes de Kudos

| Campo | Detalle |
| :--- | :--- |
| **Prioridad** | Alta |
| **Propósito** | Verificar que los Dockerfiles de Frontend (React), Producer API y Consumer Worker (Spring) utilicen construcción multietapa para eliminar dependencias de compilación. |

### Contexto y Preparación
* **Precondiciones:** Acceso a repositorios y Docker instalado.
* **Datos de Prueba:** Dockerfiles de Frontend (React v19.2), Producer API (Java 17) y Consumer Worker (Java 17).

### Cuerpo de la Prueba
1. Abrir el Dockerfile del componente a validar.
2. Confirmar la existencia de al menos dos instrucciones `FROM` (ej: `AS build` y `AS runtime`).
3. Verificar que solo se copien los artefactos finales (`dist/` o `.jar`) mediante `COPY --from`.
4. Construir imagen: `docker build -t sofkianos-test .`
5. Inspeccionar tamaño: `docker history sofkianos-test`.

**Resultado Esperado:** Imágenes ligeras sin herramientas de compilación, optimizadas para Rolling Update.

#### Escenario (Gherkin)
> **Dado** que se necesita construir las imágenes para Frontend y microservicios  
> **Cuando** se ejecutan las instrucciones del Dockerfile  
> **Entonces** el proceso utiliza Multi-stage builds, garantizando que la imagen final no contenga SDKs.

---

##  TC_002: Ejecución de Contenedores con Usuario No-Root (appuser)

| Campo | Detalle |
| :--- | :--- |
| **Prioridad** | Alta |
| **Propósito** | Garantizar que los servicios no se ejecuten con privilegios de administrador para mitigar riesgos de seguridad. |

### Cuerpo de la Prueba
1. Identificar usuario activo: `docker exec [ID_CONTENEDOR] whoami`.
2. Intentar crear archivo en ruta protegida: `docker exec [ID_CONTENEDOR] touch /root/test.txt`.

**Resultado Esperado:** `whoami` devuelve `appuser`. El intento de escritura en `/root/` es denegado ("Permission denied").

#### Escenario (Gherkin)
> **Dado** una imagen Docker construida  
> **Cuando** el contenedor se despliega en SofkianOS-MVP  
> **Entonces** el proceso principal se ejecuta exclusivamente bajo el usuario `appuser`.

---

## TC_003: Bloqueo de PR por Vulnerabilidades Críticas en Pipeline CI/CD

| Campo | Detalle |
| :--- | :--- |
| **Prioridad** | Alta |
| **Propósito** | Bloquear el despliegue de imágenes con vulnerabilidades que comprometan datos (GDPR). |

### Cuerpo de la Prueba
1. Introducir una dependencia vulnerable en una rama.
2. Abrir Pull Request (PR) hacia `main`.
3. Monitorear el paso **Security Scan (Trivy)** en GitHub Actions.

**Resultado Esperado:** El pipeline falla, el paso de Trivy arroja error y se deshabilita el botón de "Merge".

#### Escenario (Gherkin)
> **Dado** un nuevo Pull Request en GitHub  
> **Cuando** se dispara el workflow y Trivy detecta riesgos CRÍTICOS o ALTOS  
> **Entonces** el pipeline falla y se bloquea el merge.

---

##  TC_004: Continuidad de Pipeline con Vulnerabilidades de Bajo Impacto

| Campo | Detalle |
| :--- | :--- |
| **Prioridad** | Media |
| **Propósito** | Verificar que el pipeline no se detenga ante riesgos menores, manteniendo la agilidad del MVP. |

### Cuerpo de la Prueba
1. Realizar commit con vulnerabilidades "MEDIUM" o "LOW".
2. Esperar finalización de Trivy.
3. Revisar estado del check en GitHub.

**Resultado Esperado:** El pipeline finaliza con éxito (check verde). Los logs muestran las vulnerabilidades como advertencias.

#### Escenario (Gherkin)
> **Dado** un nuevo Pull Request  
> **Cuando** se detectan vulnerabilidades MEDIAS o BAJAS  
> **Entonces** se reportan los hallazgos en logs y el pipeline finaliza sin bloquear el merge.

---

##  BLOQUE 5: TRAZABILIDAD DE REQUISITOS (RTM)

| ID Caso | Requisito (US-001) | Objetivo de Negocio |
| :--- | :--- | :--- |
| **TC_001** | Criterio 1: Multi-stage | Optimización de recursos y eficiencia. |
| **TC_002** | Criterio 2: Usuario appuser | Cumplimiento de estándares y seguridad física. |
| **TC_003** | Criterio 3: Trivy (Bloqueo) | Protección de datos (GDPR/Ley 1581). |
| **TC_004** | Criterio 3: Trivy (Logs) | Visibilidad de riesgos sin impactar velocidad. |