## US-001: Refactorización y estandarización de los Dockerfile para seguridad y eficiencia

### 1. Definición de la HU

**Como** persona desarrolladora
**Quiero** refactorizar y estandarizar los Dockerfile de todos los servicios
**Para** garantizar imágenes seguras, eficientes y alineadas con las mejores prácticas

### 2. Especificaciones de Arquitectura y Despliegue

* **Capa de Clean Architecture:** Infraestructura
* **Patrón Aplicado:** Adaptadores de infraestructura, separación de build y runtime
* **Estrategia de Despliegue:** Rolling Update

### 3. Matriz de Calidad INVEST

| Criterio | Puntuación (0-3) | Justificación de la nota |
| :--- | :---: | :--- |
| **Independent** | 3 | Puede ejecutarse sin depender de otras historias |
| **Negotiable** | 3 | El enfoque de refactorización puede adaptarse |
| **Valuable** | 3 | Mejora seguridad y performance del producto |
| **Estimable** | 3 | Alcance claro y medible |
| **Small** | 2 | Puede requerir dividirse si hay muchos servicios |
| **Testable** | 3 | Se puede validar con linters y pruebas de build |

### 4. Validación (Gherkin)

* **Escenario:** Build seguro y eficiente de imágenes Docker
  * **Dado** que existen Dockerfile legacy en los servicios
  * **Cuando** se refactorizan y aplican buenas prácticas de seguridad y eficiencia
  * **Entonces** las imágenes resultantes pasan escaneo de vulnerabilidades y cumplen con los tiempos de build esperados

### 5. Definición de Hecho (DoD)

* [ ] Instalada en entornos de pre-producción.

* [ ] Pruebas de humo (Smoke Tests) superadas.
* [ ] Pruebas de regresión completadas.
* [ ] Dockerfile validados con linters y escáner de vulnerabilidades.

---

## US-002 Mejorar y versionar los archivos docker-compose para ambientes dev, test y prod

### 1. Definición de la HU

**Como** persona de operaciones
**Quiero** mejorar y versionar los archivos docker-compose para cada ambiente
**Para** facilitar despliegues consistentes y reproducibles en dev, test y prod

### 2. Especificaciones de Arquitectura y Despliegue

* **Capa de Clean Architecture:** Infraestructura
* **Patrón Aplicado:** Adaptadores de configuración, separación de ambientes
* **Estrategia de Despliegue:** Rolling Update

### 3. Matriz de Calidad INVEST

| Criterio | Puntuación (0-3) | Justificación de la nota |
| :--- | :---: | :--- |
| **Independent** | 3 | Cada ambiente puede mejorarse de forma aislada |
| **Negotiable** | 3 | Se puede ajustar la estructura de los archivos |
| **Valuable** | 3 | Reduce errores y acelera despliegues |
| **Estimable** | 3 | Alcance claro y delimitado |
| **Small** | 3 | Puede completarse en un sprint |
| **Testable** | 3 | Se valida con despliegues en cada ambiente |

### 4. Validación (Gherkin)

* **Escenario:** Despliegue reproducible por ambiente
  * **Dado** que existen archivos docker-compose desactualizados
  * **Cuando** se actualizan y versionan para dev, test y prod
  * **Entonces** los servicios se despliegan correctamente en cada ambiente sin errores de configuración

### 5. Definición de Hecho (DoD)

* [ ] Instalada en entornos de pre-producción.

* [ ] Pruebas de humo (Smoke Tests) superadas.
* [ ] Pruebas de regresión completadas.
* [ ] Versionado y documentación de los archivos docker-compose.

---

## US-009 Migración a Clean Architecture y Estabilización: consumer-worker

### 1. Diagnóstico del Monolito Activo

* **Problema detectado:** Aunque el `consumer-worker` aloja el endpoint `GET` (logrando CQRS a nivel microservicio), su lógica interna sigue fuertemente acoplada. El `KudoServiceImpl` depende directamente de Entidades JPA (`com.sofkianos.consumer.entity.Kudo`) y de modelos de paginación de Spring Data (`Page`, `Pageable`).
* **Contexto:** Al igual que ocurría históricamente con el `producer-api`, este acoplamiento viola la Inversión de Dependencias, dificultando aislar las reglas de negocio en pruebas unitarias puras y atando el núcleo al framework de BD.
* **Riesgo:** Alta fragilidad ante cambios de persistencia y baja mantenibilidad a largo plazo.

### 2. Historia de Usuario de Migración

**Como** Arquitecto de Software
**Quiero** extraer la lógica de persistencia, consultas y mensajería a casos de uso y puertos independientes (Clean Architecture) dentro del `consumer-worker`
**Para** desacoplar el dominio de la infraestructura (Spring Data / RabbitMQ) y estandarizar este servicio bajo el mismo nivel de calidad del `producer-api`.

### 3. Diseño de Clean Architecture

* **Capa Destino:** Domain (entidades puras `Kudo`, puertos in/out), Application (casos de uso segregados: `KudoQueryUseCase`, `KudoCommandUseCase`), Infrastructure (adaptadores web/rest, adaptadores JPA, listeners RabbitMQ).
* **Puertos previstos:** Interfaz `KudoPersistencePort` retornando tipos primitivos/dominio puro, eliminando referencias a `org.springframework.data.domain.Page`.
* **Patrón de Despliegue:** Refactorización interna progresiva asegurando la inmutabilidad de la API actual.

### 4. Matriz INVEST

| Criterio        | Puntos (0-3) | Observación |
| :---            | :---:        | :--- |
| **Independent** | 3            | Aislable del flujo principal de creación de datos (Commands). |
| **Negotiable**  | 3            | La migración de atributos paginados puede implementarse gradualmente. |
| **Valuable**    | 3            | Termina de desvincular el Producer, mejorando el escalado en lectura independiente del escalado de escritura. |
| **Estimable**   | 3            | El esfuerzo es predecible tomando el código del Producer como referencia. |
| **Small**       | 2            | Migrar BD y Endpoint tomará un sub-ciclo entero o sprint mediano. |
| **Testable**    | 3            | La capa Testcontainers ya existente garantiza pruebas unitarias transparentes. |

### 5. Criterios de Aceptación (Gherkin)

* **Escenario:** Inversión de Control exitosa y exposición de lectura (Query Endpoint)
  * **Dado** que el `consumer-worker` ha sido reestructurado hacia núcleos limpios (`domain`/`application`)
  * **Cuando** el usuario ejecuta un `GET` a las rutas de paginación (`/api/v1/kudos?page=0&size=20`) sobre este microservicio
  * **Entonces** la aplicación procesa la petición a través de sus nuevos RestControllers delegados a casos de uso In/Out y responde con el listado idéntico al provisto por el anterior monolito.

### 6. Definición de Hecho (DoD)

* [x] Directivas de JPA totalmente aisladas en los adaptadores *driven/out*.
* [x] Endpoint `GET` funcional y consumido exitosamente.
* [x] Pipeline CI y Testcontainers ejecutándose de manera verde con el nuevo set de tests unitarios del Core.
* [x] Código migrado respeta estricta nomenclatura y carpetas de Clean Architecture (`domain`, `application`, `infrastructure`).
