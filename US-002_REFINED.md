## US-002 Mejorar y versionar los archivos docker-compose para ambientes dev y test

### 1. Definición de la HU

**Como** persona de operaciones / DevOps
**Quiero** mejorar y versionar los archivos docker-compose aislando la configuración de cada ambiente
**Para** facilitar despliegues consistentes, seguros y con un orden de encendido confiable en dev y test.

### 2. Especificaciones de Arquitectura y Despliegue

- **Capa de Clean Architecture:** Infraestructura
- **Patrón Aplicado:** Adaptadores de configuración, separación de ambientes
- **Estrategia de Despliegue:** Rolling Update

### 3. Matriz de Calidad INVEST

| Criterio        | Puntuación (0-3) | Justificación de la nota                                                       |
| :-------------- | :--------------: | :----------------------------------------------------------------------------- |
| **Independent** |        3         | Cada ambiente puede mejorarse de forma aislada sin afectar lógica de negocio.  |
| **Negotiable**  |        3         | Se ajustó el alcance para excluir límites de CPU/RAM al no ser producción.     |
| **Valuable**    |        3         | Reduce errores de despliegue y previene exposición de credenciales en código.  |
| **Estimable**   |        3         | Alcance claro: implementar healthchecks y extraer variables a un `.env`.       |
| **Small**       |        3         | Puede completarse holgadamente en un sprint al ser solo dos ambientes espejo.  |
| **Testable**    |        3         | Se valida automatizando el levante y verificando el orden de los healthchecks. |

### 4. Validación (Gherkin)

- **Escenario 1: Gestión segura de credenciales mediante `.env`**
  - **Given** que se requiere desplegar la infraestructura (PostgreSQL, RabbitMQ)
  - **When** se leen las configuraciones en el `docker-compose.yml`
  - **Then** las credenciales de BD, credenciales de RabbitMQ, URL de conexión y nombre de base de datos se inyectan dinámicamente desde un archivo `.env`, garantizando que no existan valores _hardcodeados_.

- **Escenario 2: Resiliencia en el orden de encendido (Healthchecks)**
  - **Given** un despliegue desde cero de todos los contenedores
  - **When** se ejecuta el comando de inicialización
  - **Then** los microservicios (`producer-api`, `consumer-worker`) esperan explícitamente a que PostgreSQL y RabbitMQ reporten el estado `service_healthy` antes de intentar conectarse.

- **Escenario 3: Paridad de ambiente de Test**
  - **Given** que se necesita ejecutar pruebas en un entorno aislado
  - **When** se levanta el archivo docker-compose del ambiente de `test`
  - **Then** se despliega una réplica exacta del ambiente de `dev` (sin configuraciones extra de límites de hardware ni mocks adicionales).

### 5. Definición de Hecho (DoD)

- [ ] Instalada en entornos de pre-producción (Dev/Test).
- [ ] Pruebas de humo (Smoke Tests) superadas verificando la comunicación fluida entre contenedores.
- [ ] Pruebas de regresión completadas probando la caída y recuperación de contenedores (resiliencia).
- [ ] Versionado y documentación de los archivos docker-compose.
- [ ] **Agregado QA:** Archivo `.env.example` versionado en el repositorio (con valores dummy) sirviendo de plantilla para los desarrolladores.
