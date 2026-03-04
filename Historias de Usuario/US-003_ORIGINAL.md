## US-003 Migración a Clean Architecture y Estabilización: consumer-worker

### 1. Diagnóstico del Monolito Activo

- **Problema detectado:** Aunque el `consumer-worker` aloja el endpoint `GET` (logrando CQRS a nivel microservicio), su lógica interna sigue fuertemente acoplada. El `KudoServiceImpl` depende directamente de Entidades JPA (`com.sofkianos.consumer.entity.Kudo`) y de modelos de paginación de Spring Data (`Page`, `Pageable`).
- **Contexto:** Al igual que ocurría históricamente con el `producer-api`, este acoplamiento viola la Inversión de Dependencias, dificultando aislar las reglas de negocio en pruebas unitarias puras y atando el núcleo al framework de BD.
- **Riesgo:** Alta fragilidad ante cambios de persistencia y baja mantenibilidad a largo plazo.

### 2. Historia de Usuario de Migración

**Como** Arquitecto de Software
**Quiero** extraer la lógica de persistencia, consultas y mensajería a casos de uso y puertos independientes (Clean Architecture) dentro del `consumer-worker`
**Para** desacoplar el dominio de la infraestructura (Spring Data / RabbitMQ) y estandarizar este servicio bajo el mismo nivel de calidad del `producer-api`.

### 3. Diseño de Clean Architecture

- **Capa Destino:** Domain (entidades puras `Kudo`, puertos in/out), Application (casos de uso segregados: `KudoQueryUseCase`, `KudoCommandUseCase`), Infrastructure (adaptadores web/rest, adaptadores JPA, listeners RabbitMQ).
- **Puertos previstos:** Interfaz `KudoPersistencePort` retornando tipos primitivos/dominio puro, eliminando referencias a `org.springframework.data.domain.Page`.
- **Patrón de Despliegue:** Refactorización interna progresiva asegurando la inmutabilidad de la API actual.

### 4. Matriz INVEST

| Criterio        | Puntos (0-3) | Observación                                                                                                   |
| :-------------- | :----------: | :------------------------------------------------------------------------------------------------------------ |
| **Independent** |      3       | Aislable del flujo principal de creación de datos (Commands).                                                 |
| **Negotiable**  |      3       | La migración de atributos paginados puede implementarse gradualmente.                                         |
| **Valuable**    |      3       | Termina de desvincular el Producer, mejorando el escalado en lectura independiente del escalado de escritura. |
| **Estimable**   |      3       | El esfuerzo es predecible tomando el código del Producer como referencia.                                     |
| **Small**       |      2       | Migrar BD y Endpoint tomará un sub-ciclo entero o sprint mediano.                                             |
| **Testable**    |      3       | La capa Testcontainers ya existente garantiza pruebas unitarias transparentes.                                |

### 5. Criterios de Aceptación (Gherkin)

- **Escenario:** Inversión de Control exitosa y exposición de lectura (Query Endpoint)
  - **Dado** que el `consumer-worker` ha sido reestructurado hacia núcleos limpios (`domain`/`application`)
  - **Cuando** el usuario ejecuta un `GET` a las rutas de paginación (`/api/v1/kudos?page=0&size=20`) sobre este microservicio
  - **Entonces** la aplicación procesa la petición a través de sus nuevos RestControllers delegados a casos de uso In/Out y responde con el listado idéntico al provisto por el anterior monolito.

### 6. Definición de Hecho (DoD)

- [x] Directivas de JPA totalmente aisladas en los adaptadores _driven/out_.
- [x] Endpoint `GET` funcional y consumido exitosamente.
- [x] Pipeline CI y Testcontainers ejecutándose de manera verde con el nuevo set de tests unitarios del Core.
- [x] Código migrado respeta estricta nomenclatura y carpetas de Clean Architecture (`domain`, `application`, `infrastructure`).
