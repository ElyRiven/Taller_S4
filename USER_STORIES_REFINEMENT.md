## US-001: Implementación de Dockerfiles Multi-stage, Appuser y Escaneo de Seguridad en PR

### 1. Definición de la HU

**Como** persona desarrolladora
**Quiero** estandarizar los Dockerfiles de Frontend, Producer API y Consumer Worker implementando multi-stage builds, usuarios no-root y análisis de vulnerabilidades automatizado
**Para** garantizar que solo se desplieguen imágenes seguras, eficientes y optimizadas en los entornos de SofkianOS-MVP.

### 2. Especificaciones de Arquitectura y Despliegue

* **Capa de Clean Architecture:** Infraestructura
* **Patrón Aplicado:** Adaptadores de infraestructura, separación de build y runtime
* **Estrategia de Despliegue:** Rolling Update

### 3. Matriz de Calidad INVEST

| Criterio | Puntuación (0-3) | Justificación de la nota |
| :--- | :---: | :--- |
| **Independent** | 3 | Se ejecuta 100% a nivel de repositorio e infraestructura. |
| **Negotiable** | 3 | Se acordó el uso específico de Trivy, Github Actions y el usuario `appuser`. |
| **Valuable** | 3 | Previene la inyección de código vulnerable directamente desde la etapa de PR. |
| **Estimable** | 3 | El esfuerzo es claro al definir las herramientas exactas y las reglas de bloqueo. |
| **Small** | 3 | Alcance perfectamente acotado a los 3 componentes del MVP. |
| **Testable** | 3 | QA puede forzar imágenes vulnerables para validar el bloqueo del pipeline. |

### 4. Validación (Gherkin)

* **Escenario 1: Optimización Multi-stage**
  * **Given** que se necesita construir las imágenes para Frontend y los microservicios Backend
  * **When** se ejecutan las instrucciones del Dockerfile
  * **Then** el proceso utiliza *Multi-stage builds*, garantizando que la imagen final no contenga SDKs ni herramientas de compilación.

* **Escenario 2: Privilegios de ejecución No-Root**
  * **Given** una imagen Docker construida y lista para ejecución
  * **When** el contenedor se despliega en cualquier ambiente
  * **Then** el proceso principal se ejecuta exclusivamente bajo el usuario `appuser` (previamente creado en el Dockerfile con los permisos mínimos necesarios) y nunca como `root`.

* **Escenario 3: Bloqueo de seguridad automatizado en CI/CD**
  * **Given** un nuevo Pull Request o actualización de código en GitHub
  * **When** se dispara el workflow de GitHub Actions
  * **Then** se construye la imagen y se ejecuta un escaneo con **Trivy**, fallando el pipeline y bloqueando el merge si existen vulnerabilidades **CRÍTICAS o ALTAS**, o reportando en los logs (sin fallar) si son **MEDIAS o BAJAS**.

### 5. Definición de Hecho (DoD)

* [ ] Instalada en entornos de pre-producción.
* [ ] Pruebas de humo (Smoke Tests) superadas.
* [ ] Pruebas de regresión completadas.
* [ ] Dockerfile validados implementando *Multi-stage* y usuario `appuser`.
* [ ] **Agregado QA:** Workflow de GitHub Actions configurado con Trivy y validado su correcto bloqueo ante imágenes vulnerables.

---

## US-002 Mejorar y versionar los archivos docker-compose para ambientes dev y test

### 1. Definición de la HU

**Como** persona de operaciones / DevOps
**Quiero** mejorar y versionar los archivos docker-compose aislando la configuración de cada ambiente
**Para** facilitar despliegues consistentes, seguros y con un orden de encendido confiable en dev y test.

### 2. Especificaciones de Arquitectura y Despliegue

* **Capa de Clean Architecture:** Infraestructura
* **Patrón Aplicado:** Adaptadores de configuración, separación de ambientes
* **Estrategia de Despliegue:** Rolling Update

### 3. Matriz de Calidad INVEST

| Criterio | Puntuación (0-3) | Justificación de la nota |
| :--- | :---: | :--- |
| **Independent** | 3 | Cada ambiente puede mejorarse de forma aislada sin afectar lógica de negocio. |
| **Negotiable** | 3 | Se ajustó el alcance para excluir límites de CPU/RAM al no ser producción. |
| **Valuable** | 3 | Reduce errores de despliegue y previene exposición de credenciales en código. |
| **Estimable** | 3 | Alcance claro: implementar healthchecks y extraer variables a un `.env`. |
| **Small** | 3 | Puede completarse holgadamente en un sprint al ser solo dos ambientes espejo. |
| **Testable** | 3 | Se valida automatizando el levante y verificando el orden de los healthchecks. |

### 4. Validación (Gherkin)

* **Escenario 1: Gestión segura de credenciales mediante `.env`**
  * **Given** que se requiere desplegar la infraestructura (PostgreSQL, RabbitMQ)
  * **When** se leen las configuraciones en el `docker-compose.yml`
  * **Then** las credenciales de BD, credenciales de RabbitMQ, URL de conexión y nombre de base de datos se inyectan dinámicamente desde un archivo `.env`, garantizando que no existan valores *hardcodeados*.

* **Escenario 2: Resiliencia en el orden de encendido (Healthchecks)**
  * **Given** un despliegue desde cero de todos los contenedores
  * **When** se ejecuta el comando de inicialización
  * **Then** los microservicios (`producer-api`, `consumer-worker`) esperan explícitamente a que PostgreSQL y RabbitMQ reporten el estado `service_healthy` antes de intentar conectarse.

* **Escenario 3: Paridad de ambiente de Test**
  * **Given** que se necesita ejecutar pruebas en un entorno aislado
  * **When** se levanta el archivo docker-compose del ambiente de `test`
  * **Then** se despliega una réplica exacta del ambiente de `dev` (sin configuraciones extra de límites de hardware ni mocks adicionales).

### 5. Definición de Hecho (DoD)

* [ ] Instalada en entornos de pre-producción (Dev/Test).
* [ ] Pruebas de humo (Smoke Tests) superadas verificando la comunicación fluida entre contenedores.
* [ ] Pruebas de regresión completadas probando la caída y recuperación de contenedores (resiliencia).
* [ ] Versionado y documentación de los archivos docker-compose.
* [ ] **Agregado QA:** Archivo `.env.example` versionado en el repositorio (con valores dummy) sirviendo de plantilla para los desarrolladores.

---

# US-009 — Migración a Clean Architecture y Estabilización: consumer-worker

## 1. Diagnóstico del Monolito Activo

* **Problema detectado:** Aunque el `consumer-worker` aloja el endpoint `GET` (logrando CQRS a nivel microservicio), su lógica interna sigue fuertemente acoplada. El `KudoServiceImpl` depende directamente de Entidades JPA (`com.sofkianos.consumer.entity.Kudo`) y de modelos de paginación de Spring Data (`Page`, `Pageable`).
* **Contexto:** Al igual que ocurría históricamente con el `producer-api`, este acoplamiento viola la Inversión de Dependencias, dificultando aislar las reglas de negocio en pruebas unitarias puras y atando el núcleo al framework de BD.
* **Riesgo:** Alta fragilidad ante cambios de persistencia y baja mantenibilidad a largo plazo.

---

## 2. Historia de Usuario de Migración

**Como** Arquitecto de Software  
**Quiero** extraer la lógica de persistencia, consultas y mensajería a casos de uso y puertos independientes (Clean Architecture) dentro del `consumer-worker`  
**Para** desacoplar el dominio de la infraestructura (Spring Data / RabbitMQ) y estandarizar este servicio bajo el mismo nivel de calidad del `producer-api`.

---

## 3. Diseño de Clean Architecture

* **Capa Destino:** Domain (entidades puras `Kudo`, puertos in/out), Application (casos de uso segregados: `KudoQueryUseCase`, `KudoCommandUseCase`), Infrastructure (adaptadores web/rest, adaptadores JPA, listeners RabbitMQ).
* **Puertos previstos:** Interfaz `KudoPersistencePort` retornando tipos primitivos/dominio puro, eliminando referencias a `org.springframework.data.domain.Page`.
* **Patrón de Despliegue:** Refactorización interna progresiva asegurando la inmutabilidad de la API actual.

### 3.1 Contrato de API — Estructura JSON de Paginación

El endpoint `GET /api/v1/kudos` retornará el siguiente JSON usando el DTO `PagedKudoResponse` (desacoplado de `org.springframework.data.domain.Page`):

```json
{
  "content": [
    {
      "id": 123,
      "fromUser": "alice@sofkianos.com",
      "toUser": "bob@sofkianos.com",
      "category": "TEAMWORK",
      "message": "Great sprint delivery!",
      "createdAt": "2026-03-01T10:30:00"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 100,
  "totalPages": 5
}
```

**Clase:** `com.sofkianos.consumer.dto.PagedKudoResponse`  
**Garantía de estabilidad:** Contract Test requerido en `src/test/resources/contracts/kudos/shouldReturnPaginatedKudos.groovy`

---

## 4. Matriz INVEST

| Criterio        | Puntos (0-3) | Observación                                                                         |
| :---            |    :---:     | :---                                                                                |
| **Independent** | 3            | Aislable del flujo principal de creación de datos (Commands).                       |
| **Negotiable**  | 3            | La migración de atributos paginados puede implementarse gradualmente.               |
| **Valuable**    | 3            | Termina de desvincular el Producer, mejorando el escalado en lectura independiente. |
| **Estimable**   | 3            | El esfuerzo es predecible tomando el código del Producer como referencia.           |
| **Small**       | 2            | Migrar BD y Endpoint tomará un sub-ciclo entero o sprint mediano.                   |
| **Testable**    | 3            | La capa Testcontainers ya existente garantiza pruebas unitarias transparentes.      |

---

## 5. Criterios de Aceptación (Gherkin)

#### AC 1 (Query — Happy Path): Exposición de lectura paginada

* **Given** que el `consumer-worker` ha sido reestructurado hacia núcleos limpios (`domain`/`application`)
* **When** el usuario ejecuta un `GET /api/v1/kudos?page=0&size=20`
* **Then** la aplicación procesa la petición a través de los nuevos RestControllers delegados a casos de uso y responde con el JSON definido en el contrato `PagedKudoResponse`
* **And** el DTO no importa ninguna clase de `org.springframework.data.domain`

#### AC 2 (Command — Happy Path): Consumo exitoso de mensajes de RabbitMQ

* **Given** que existe un mensaje válido en la cola `kudos.queue`
* **When** el listener `KudosConsumer` recibe el mensaje
* **Then** delega al `KudoService.saveKudo()` y el Kudo es persistido en PostgreSQL
* **And** el mensaje recibe ACK automático de Spring AMQP

#### AC 3 (Query — Sad Path): Parámetros de paginación inválidos

* **Given** que se llama a `GET /api/v1/kudos` con parámetros inválidos (ej. `page=-1`)
* **When** el controlador procesa la solicitud
* **Then** retorna HTTP 400 con mensaje descriptivo del error

#### AC 4 (Command — Sad Path): Rechazo de mensaje inválido *(ya implementado)*

* **Given** que existe un mensaje malformado en `kudos.queue` (sin campo `from`, `to` o `message`)
* **When** el listener intenta procesarlo
* **Then** el servicio lanza `InvalidKudoException`
* **And** el mensaje es enrutado a la Dead Letter Queue `kudos.dlq`
* **And** el listener continúa procesando el siguiente mensaje sin interrupciones

#### AC 5 (Resiliencia): Consumer Worker tolerante a caída del servicio de persistencia

* **Given** que RabbitMQ sigue enviando mensajes a `kudos.queue`
* **And** PostgreSQL no está disponible temporalmente
* **When** el `consumer-worker` intenta procesar un mensaje
* **Then** la aplicación **no falla ni se detiene**
* **And** los mensajes permanecen en la cola de RabbitMQ
* **And** cuando el servicio de persistencia vuelve a estar disponible, los mensajes pendientes son procesados automáticamente sin pérdida de datos

#### AC 6 (Testing): Cobertura de integración del consumer

* **Given** que el refactor está completado
* **When** se ejecuta el pipeline de CI
* **Then** existe un test de integración que:
  * Levanta RabbitMQ con Testcontainers o EmbeddedAmqp
  * Publica un mensaje válido usando `RabbitTemplate`
  * Verifica que el Kudo fue persistido consultando `KudoRepository`
* **And** existe un test que verifica el enrutamiento a `kudos.dlq` ante mensajes inválidos

---

## 6. Definición de Hecho (DoD)

* [ ] Directivas de JPA totalmente aisladas en los adaptadores *driven/out*
* [ ] El DTO `PagedKudoResponse` no importa ninguna clase de `org.springframework.data.domain`
* [ ] Existe un Contract Test en `src/test/resources/contracts/` que valida los campos `content`, `page`, `size`, `totalElements`, `totalPages`
* [ ] Endpoint `GET` funcional y validado contra el contrato de API definido en §3.1
* [ ] DLQ `kudos.dlq` declarada en `RabbitConfig` y enlazada a `kudos.dlx`
* [ ] El `consumer-worker` no falla si PostgreSQL no está disponible; los mensajes se mantienen en cola hasta que el servicio levante
* [ ] Test de integración cubre happy path y sad path del consumer
* [ ] Pipeline CI pasa en verde con el nuevo set de tests
* [ ] Código migrado respeta nomenclatura y estructura de Clean Architecture (`domain`, `application`, `infrastructure`)