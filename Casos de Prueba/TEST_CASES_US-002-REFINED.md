# SUITE DE CASOS DE PRUEBA: ORQUESTACIÓN Y CONFIGURACIÓN (US-002)
**Proyecto:** SofkianOS-MVP  
**Enfoque:** Gestión de secretos, resiliencia y paridad de ambientes.

---

## TC_005: Inyección Dinámica de Secretos (.env)
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_005 |
| **Prioridad** | Alta |
| **Propósito** | Validar que las credenciales no estén hardcodeadas en el orquestador. |

### Pasos de Ejecución
1. Inspeccionar `docker-compose.yml` y verificar el uso de `${VARIABLES}`.
2. Ejecutar `docker-compose config` y revisar que no se expongan secretos en texto plano.
3. Levantar servicios e inspeccionar variables dentro del contenedor (`env`).

**Resultado Esperado:** Credenciales inyectadas desde `.env`, invisibles en el archivo YAML y en logs de arranque.

---

## TC_006: Control de Secuencia de Inicio (Healthchecks)
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_006 |
| **Prioridad** | Alta |
| **Propósito** | Asegurar que los microservicios esperen a PostgreSQL y RabbitMQ antes de iniciar. |

### Pasos de Ejecución
1. Apagar todos los contenedores.
2. Levantar el stack completo con `docker-compose up -d`.
3. Monitorear estados y logs de reintentos controlados con backoff (intervalos crecientes entre reintentos).
4. Simular latencia o arranque lento de una dependencia (ej. retrasar PostgreSQL 30s) y verificar que los servicios dependientes esperen con reintentos y backoff sin fallar.

**Resultado Esperado:** Los servicios inician solo cuando las dependencias están **healthy**, aplicando reintentos con backoff controlado y evitando errores de conexión temprana incluso ante latencia de arranque.

---

## TC_007: Paridad Funcional entre Ambientes Dev y Test
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_007 |
| **Prioridad** | Media |
| **Propósito** | Garantizar consistencia funcional entre ambientes. |

### Pasos de Ejecución
1. Comparar imágenes, versiones, variables de entorno y flags de configuración entre los archivos de `dev` y `test`.
2. Verificar que las variables de entorno y flags de configuración (ej. `SPRING_PROFILES_ACTIVE`, `LOG_LEVEL`, feature toggles) sean consistentes entre ambientes.
3. Levantar ambiente de test y ejecutar una prueba de humo funcional.

**Resultado Esperado:** Ambos ambientes operan con la misma arquitectura, comportamiento, variables de entorno y flags de configuración.

---

## TC_008: Disponibilidad de Plantilla .env.example
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_008 |
| **Prioridad** | Baja |
| **Propósito** | Proveer una guía segura de configuración local para desarrolladores. |

### Pasos de Ejecución
1. Verificar existencia de `.env.example` en la raíz.
2. Confirmar que no contenga credenciales reales y que esté en el `.gitignore`.
3. Buscar en todos los archivos `docker-compose*.yml` que no exista referencia directa a `.env.example` como `env_file`.
4. Revisar los workflows de CI/CD (`.github/workflows/`) y confirmar que `.env.example` no sea utilizado como fuente de configuración en ningún paso del pipeline.

**Resultado Esperado:** El archivo sirve como documentación técnica estandarizada, sin ser referenciado accidentalmente en ningún `docker-compose` ni pipeline de CI/CD.

---

## 📊 MATRIZ DE TRAZABILIDAD – US-002
| ID Caso | Requisito | Objetivo de Negocio |
| :--- | :--- | :--- |
| TC_005 | Gestión de secretos | Cumplimiento normativo |
| TC_006 | Resiliencia | Estabilidad del sistema |
| TC_007 | Paridad | Consistencia en QA |
| TC_008 | .env.example | Estandarización técnica |