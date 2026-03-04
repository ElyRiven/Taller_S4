# 🛠️ SUITE DE CASOS DE PRUEBA: ORQUESTACIÓN Y CONFIGURACIÓN (US-002)
**Proyecto:** SofkianOS-MVP  
**Enfoque:** Validación de infraestructura, gestión de secretos y resiliencia.

---

##  TC_005: Inyección Dinámica de Secretos (.env)

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_005 |
| **Prioridad** | Alta |
| **Propósito** | Validar que ninguna credencial de PostgreSQL o RabbitMQ esté en texto plano en los archivos YAML, cumpliendo con la Ley 1581. |

###  Pasos de Ejecución
1. Abrir `docker-compose.yml` y buscar las secciones `environment`.
2. Confirmar que los valores sean variables (ej: `${DB_PASSWORD}`) y no texto plano.
3. Ejecutar `docker-compose config` para verificar la resolución de variables.
4. Levantar servicios: `docker-compose up -d`.
5. Inspeccionar entorno en contenedor: `docker exec [ID_DB] env`.

> **Resultado Esperado:** El sistema resuelve las variables desde el `.env`. No se encuentran contraseñas hardcodeadas en el archivo YAML.

---

## TC_006: Control de Secuencia de Inicio (Healthchecks)

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_006 |
| **Prioridad** | Alta |
| **Propósito** | Garantizar que los microservicios (Producer/Consumer) esperen a que la DB y el Broker estén listos para recibir conexiones. |

###  Pasos de Ejecución
1. Asegurar que los contenedores estén apagados.
2. Ejecutar `docker-compose up -d`.
3. Monitorear el estado: `docker-compose ps`.
4. Verificar logs: Confirmar que las APIs inician su conexión solo después del mensaje "database system is ready".

> **Resultado Esperado:** Los microservicios esperan el estado `healthy` de las dependencias, evitando fallos de "Connection Refused".

---

## TC_007: Paridad Funcional entre Ambientes Dev y Test

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_007 |
| **Prioridad** | Media |
| **Propósito** | Asegurar que el ambiente de Test sea una réplica exacta de Dev para evitar errores de inconsistencia. |

###  Pasos de Ejecución
1. Comparar imágenes y versiones en `docker-compose.dev.yml` y `docker-compose.test.yml`.
2. Levantar ambiente de test: `docker-compose -f docker-compose.test.yml up -d`.
3. Realizar prueba de humo (envío de Kudo) desde el Frontend de Test.

> **Resultado Esperado:** Ambos ambientes operan con las mismas versiones de software (Spring v3.4.5, etc.) y arquitectura.

---

##  TC_008: Disponibilidad de Plantilla .env.example

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_008 |
| **Prioridad** | Baja |
| **Propósito** | Validar que el proyecto incluya una guía de configuración local segura para nuevos desarrolladores. |

###  Pasos de Ejecución
1. Listar archivos en la raíz del proyecto: `ls -a`.
2. Verificar la existencia y versionamiento de `.env.example`.
3. Validar que el contenido incluya solo valores descriptivos o "dummy".

> **Resultado Esperado:** El archivo existe como documentación técnica y no contiene credenciales reales de ningún entorno.

---

##  TRAZABILIDAD (RTM)

| ID Caso | Requisito (US-002) | Objetivo de Negocio |
| :--- | :--- | :--- |
| **TC_005** | Escenario: Gestión segura | Seguridad y cumplimiento normativo. |
| **TC_006** | Escenario: Resiliencia | Estabilidad del sistema de Kudos. |
| **TC_007** | Escenario: Paridad | Consistencia en QA y despliegues. |
| **TC_008** | DoD: Archivo .env.example | Estandarización de la cultura Sofkiana. |