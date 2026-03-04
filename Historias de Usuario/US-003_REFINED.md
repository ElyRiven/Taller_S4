# US-003 — Migración a Clean Architecture y Estabilización: consumer-worker

## Justificación: Estado WIP por Limitaciones de Tiempo

La migración del `consumer-worker` hacia una arquitectura limpia representa un esfuerzo significativo, ya que implica desacoplar múltiples capas y dependencias históricas del monolito original. Dada la complejidad técnica y la necesidad de mantener la estabilidad operativa del servicio, el equipo ha decidido abordar esta historia de usuario como un **Work In Progress (WIP)**. Esta decisión responde principalmente a la restricción de tiempo disponible en el ciclo actual, lo que hace inviable completar la migración total de arquitectura y la cobertura de todos los criterios de aceptación en un solo sprint.

El enfoque WIP permite avanzar de manera incremental, priorizando la refactorización de los componentes más críticos y asegurando entregables funcionales y testeables en cada iteración. Así, se minimizan riesgos de regresión y se facilita la validación continua, sin comprometer la calidad ni la disponibilidad del servicio. Esta estrategia también habilita la retroalimentación temprana y la adaptación del plan conforme se identifican nuevos retos técnicos durante la migración.

## 1. Diagnóstico del Monolito Activo

- **Problema detectado:** Aunque el `consumer-worker` aloja el endpoint `GET` (logrando CQRS a nivel microservicio), su lógica interna sigue fuertemente acoplada. El `KudoServiceImpl` depende directamente de Entidades JPA (`com.sofkianos.consumer.entity.Kudo`) y de modelos de paginación de Spring Data (`Page`, `Pageable`).
- **Contexto:** Al igual que ocurría históricamente con el `producer-api`, este acoplamiento viola la Inversión de Dependencias, dificultando aislar las reglas de negocio en pruebas unitarias puras y atando el núcleo al framework de BD.
- **Riesgo:** Alta fragilidad ante cambios de persistencia y baja mantenibilidad a largo plazo.

---

## 2. Historia de Usuario de Migración

**Como** Arquitecto de Software  
**Quiero** extraer la lógica de persistencia, consultas y mensajería a casos de uso y puertos independientes (Clean Architecture) dentro del `consumer-worker`  
**Para** desacoplar el dominio de la infraestructura (Spring Data / RabbitMQ) y estandarizar este servicio bajo el mismo nivel de calidad del `producer-api`.

---

## 3. Diseño de Clean Architecture

- **Capa Destino:** Domain (entidades puras `Kudo`, puertos in/out), Application (casos de uso segregados: `KudoQueryUseCase`, `KudoCommandUseCase`), Infrastructure (adaptadores web/rest, adaptadores JPA, listeners RabbitMQ).
- **Puertos previstos:** Interfaz `KudoPersistencePort` retornando tipos primitivos/dominio puro, eliminando referencias a `org.springframework.data.domain.Page`.
- **Patrón de Despliegue:** Refactorización interna progresiva asegurando la inmutabilidad de la API actual.

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
| :-------------- | :----------: | :---------------------------------------------------------------------------------- |
| **Independent** |      3       | Aislable del flujo principal de creación de datos (Commands).                       |
| **Negotiable**  |      3       | La migración de atributos paginados puede implementarse gradualmente.               |
| **Valuable**    |      3       | Termina de desvincular el Producer, mejorando el escalado en lectura independiente. |
| **Estimable**   |      3       | El esfuerzo es predecible tomando el código del Producer como referencia.           |
| **Small**       |      2       | Migrar BD y Endpoint tomará un sub-ciclo entero o sprint mediano.                   |
| **Testable**    |      3       | La capa Testcontainers ya existente garantiza pruebas unitarias transparentes.      |

---

## 5. Criterios de Aceptación (Gherkin)

#### AC 1 (Query — Happy Path): Exposición de lectura paginada

- **Given** que el `consumer-worker` ha sido reestructurado hacia núcleos limpios (`domain`/`application`)
- **When** el usuario ejecuta un `GET /api/v1/kudos?page=0&size=20`
- **Then** la aplicación procesa la petición a través de los nuevos RestControllers delegados a casos de uso y responde con el JSON definido en el contrato `PagedKudoResponse`
- **And** el DTO no importa ninguna clase de `org.springframework.data.domain`

#### AC 2 (Command — Happy Path): Consumo exitoso de mensajes de RabbitMQ

- **Given** que existe un mensaje válido en la cola `kudos.queue`
- **When** el listener `KudosConsumer` recibe el mensaje
- **Then** delega al `KudoService.saveKudo()` y el Kudo es persistido en PostgreSQL
- **And** el mensaje recibe ACK automático de Spring AMQP

#### AC 3 (Query — Sad Path): Parámetros de paginación inválidos

- **Given** que se llama a `GET /api/v1/kudos` con parámetros inválidos (ej. `page=-1`)
- **When** el controlador procesa la solicitud
- **Then** retorna HTTP 400 con mensaje descriptivo del error

#### AC 4 (Command — Sad Path): Rechazo de mensaje inválido _(ya implementado)_

- **Given** que existe un mensaje malformado en `kudos.queue` (sin campo `from`, `to` o `message`)
- **When** el listener intenta procesarlo
- **Then** el servicio lanza `InvalidKudoException`
- **And** el mensaje es enrutado a la Dead Letter Queue `kudos.dlq`
- **And** el listener continúa procesando el siguiente mensaje sin interrupciones

#### AC 5 (Resiliencia): Consumer Worker tolerante a caída del servicio de persistencia

- **Given** que RabbitMQ sigue enviando mensajes a `kudos.queue`
- **And** PostgreSQL no está disponible temporalmente
- **When** el `consumer-worker` intenta procesar un mensaje
- **Then** la aplicación **no falla ni se detiene**
- **And** los mensajes permanecen en la cola de RabbitMQ
- **And** cuando el servicio de persistencia vuelve a estar disponible, los mensajes pendientes son procesados automáticamente sin pérdida de datos

#### AC 6 (Testing): Cobertura de integración del consumer

- **Given** que el refactor está completado
- **When** se ejecuta el pipeline de CI
- **Then** existe un test de integración que:
  - Levanta RabbitMQ con Testcontainers o EmbeddedAmqp
  - Publica un mensaje válido usando `RabbitTemplate`
  - Verifica que el Kudo fue persistido consultando `KudoRepository`
- **And** existe un test que verifica el enrutamiento a `kudos.dlq` ante mensajes inválidos

---

## 6. Definición de Hecho (DoD)

- [ ] Directivas de JPA totalmente aisladas en los adaptadores _driven/out_
- [ ] El DTO `PagedKudoResponse` no importa ninguna clase de `org.springframework.data.domain`
- [ ] Existe un Contract Test en `src/test/resources/contracts/` que valida los campos `content`, `page`, `size`, `totalElements`, `totalPages`
- [ ] Endpoint `GET` funcional y validado contra el contrato de API definido en §3.1
- [ ] DLQ `kudos.dlq` declarada en `RabbitConfig` y enlazada a `kudos.dlx`
- [ ] El `consumer-worker` no falla si PostgreSQL no está disponible; los mensajes se mantienen en cola hasta que el servicio levante
- [ ] Test de integración cubre happy path y sad path del consumer
- [ ] Pipeline CI pasa en verde con el nuevo set de tests
- [ ] Código migrado respeta nomenclatura y estructura de Clean Architecture (`domain`, `application`, `infrastructure`)
