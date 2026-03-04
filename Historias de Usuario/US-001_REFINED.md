## US-001: Implementación de Dockerfiles Multi-stage, Appuser y Escaneo de Seguridad en PR

### 1. Definición de la HU

**Como** persona desarrolladora
**Quiero** estandarizar los Dockerfiles de Frontend, Producer API y Consumer Worker implementando multi-stage builds, usuarios no-root y análisis de vulnerabilidades automatizado
**Para** garantizar que solo se desplieguen imágenes seguras, eficientes y optimizadas en los entornos de SofkianOS-MVP.

### 2. Especificaciones de Arquitectura y Despliegue

- **Capa de Clean Architecture:** Infraestructura
- **Patrón Aplicado:** Adaptadores de infraestructura, separación de build y runtime
- **Estrategia de Despliegue:** Rolling Update

### 3. Matriz de Calidad INVEST

| Criterio        | Puntuación (0-3) | Justificación de la nota                                                          |
| :-------------- | :--------------: | :-------------------------------------------------------------------------------- |
| **Independent** |        3         | Se ejecuta 100% a nivel de repositorio e infraestructura.                         |
| **Negotiable**  |        3         | Se acordó el uso específico de Trivy, Github Actions y el usuario `appuser`.      |
| **Valuable**    |        3         | Previene la inyección de código vulnerable directamente desde la etapa de PR.     |
| **Estimable**   |        3         | El esfuerzo es claro al definir las herramientas exactas y las reglas de bloqueo. |
| **Small**       |        3         | Alcance perfectamente acotado a los 3 componentes del MVP.                        |
| **Testable**    |        3         | QA puede forzar imágenes vulnerables para validar el bloqueo del pipeline.        |

### 4. Validación (Gherkin)

- **Escenario 1: Optimización Multi-stage**
  - **Given** que se necesita construir las imágenes para Frontend y los microservicios Backend
  - **When** se ejecutan las instrucciones del Dockerfile
  - **Then** el proceso utiliza _Multi-stage builds_, garantizando que la imagen final no contenga SDKs ni herramientas de compilación.

- **Escenario 2: Privilegios de ejecución No-Root**
  - **Given** una imagen Docker construida y lista para ejecución
  - **When** el contenedor se despliega en cualquier ambiente
  - **Then** el proceso principal se ejecuta exclusivamente bajo el usuario `appuser` (previamente creado en el Dockerfile con los permisos mínimos necesarios) y nunca como `root`.

- **Escenario 3: Bloqueo de seguridad automatizado en CI/CD**
  - **Given** un nuevo Pull Request o actualización de código en GitHub
  - **When** se dispara el workflow de GitHub Actions
  - **Then** se construye la imagen y se ejecuta un escaneo con **Trivy**, fallando el pipeline y bloqueando el merge si existen vulnerabilidades **CRÍTICAS o ALTAS**, o reportando en los logs (sin fallar) si son **MEDIAS o BAJAS**.

### 5. Definición de Hecho (DoD)

- [ ] Instalada en entornos de pre-producción.
- [ ] Pruebas de humo (Smoke Tests) superadas.
- [ ] Pruebas de regresión completadas.
- [ ] Dockerfile validados implementando _Multi-stage_ y usuario `appuser`.
- [ ] **Agregado QA:** Workflow de GitHub Actions configurado con Trivy y validado su correcto bloqueo ante imágenes vulnerables.
