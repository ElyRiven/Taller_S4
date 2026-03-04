# SUITE DE CASOS DE PRUEBA: MIGRACIÓN CLEAN ARCHITECTURE (US-003)
**Proyecto:** SofkianOS-MVP | Componente: **Consumer-Worker** **Enfoque:** Desacoplamiento de framework, resiliencia de mensajería y persistencia.

---

## TC_009: Validación de Desacoplamiento de Paginación (Capa Application/Domain)

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_009 |
| **Prioridad** | Alta |
| **Propósito** | Verificar que el núcleo de la aplicación y los DTOs no dependan de clases específicas de Spring Data (Page, Pageable). |

### Pasos de Ejecución
1. Inspeccionar los imports de la clase `PagedKudoResponse`.
2. Verificar la ausencia de `org.springframework.data.domain.Page`.
3. Ejecutar compilación: `./gradlew build` o `./mvn clean compile`.
4. Confirmar que la conversión de Entidad JPA a Dominio ocurra solo en la capa de **Infrastructure**.

> **Resultado Esperado:** Compilación exitosa. El DTO solo utiliza tipos primitivos o clases propias del dominio, cumpliendo con la Inversión de Dependencias.

---

##  TC_010: Flujo de Persistencia de Kudos vía RabbitMQ

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_010 |
| **Prioridad** | Alta |
| **Propósito** | Validar que el listener de RabbitMQ delegue la persistencia correctamente al puerto de salida hacia PostgreSQL. |



### Pasos de Ejecución
1. Publicar un mensaje JSON válido en la cola `kudos.queue`.
2. Monitorear logs del `consumer-worker`.
3. Consultar PostgreSQL: `SELECT * FROM kudos WHERE from_user = 'alice@sofkianos.com'`.
4. Verificar el **ACK** en el panel de RabbitMQ.

> **Resultado Esperado:** El Kudo se registra en la base de datos y el mensaje es procesado exitosamente (desaparece de la cola).

---

## TC_011: Manejo de Errores y Enrutamiento a Dead Letter Queue (DLQ)

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_011 |
| **Prioridad** | Media |
| **Propósito** | Confirmar que los mensajes malformados no detienen el flujo y son aislados en la DLQ. |

###  Pasos de Ejecución
1. Enviar un mensaje JSON inválido (sin campos obligatorios) a la cola principal.
2. Verificar que se lance `InvalidKudoException`.
3. Comprobar que el mensaje se mueva automáticamente a `kudos.dlq`.
4. Enviar un mensaje válido inmediatamente después para probar continuidad.

> **Resultado Esperado:** El mensaje inválido termina en la DLQ. El servicio sigue operativo para procesar mensajes válidos posteriores.

---

## 🛡️ TC_012: Resiliencia ante Indisponibilidad de Base de Datos

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_012 |
| **Prioridad** | Alta |
| **Propósito** | Asegurar que no se pierdan Kudos si PostgreSQL falla, manteniendo los mensajes en la cola. |

### Pasos de Ejecución
1. Detener el contenedor de PostgreSQL intencionalmente.
2. Enviar 5 mensajes de Kudos a la cola.
3. Verificar que el servicio no haga "crash" (se mantenga reintentando).
4. Iniciar PostgreSQL y observar el procesamiento automático.

> **Resultado Esperado:** Los mensajes permanecen en RabbitMQ mientras la BD está caída y se persisten automáticamente al restaurar la conexión.

---

## TC_013: Validación de Contrato de API (Contract Testing)

| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_013 |
| **Prioridad** | Media |
| **Propósito** | Verificar que el JSON de salida cumpla estrictamente con el contrato definido para el Frontend. |

### Pasos de Ejecución
1. Ejecutar tests de contrato en `src/test/resources/contracts/`.
2. Validar presencia de campos: `content`, `page`, `size`, `totalElements`.
3. Alterar un campo en el DTO y confirmar que el test falle (validación de robustez del test).

> **Resultado Esperado:** El test pasa si la estructura JSON coincide con la especificación técnica §3.1.

---

## MATRIZ DE TRAZABILIDAD (RTM)

| ID Caso | Requisito (US-003) | Objetivo de Negocio |
| :--- | :--- | :--- |
| **TC_009** | AC 1: Desacoplamiento | Mantenibilidad y evolución técnica. |
| **TC_010** | AC 2: Consumo exitoso | Asegurar que los reconocimientos lleguen a la BD. |
| **TC_011** | AC 4: Manejo de inválidos | Integridad de datos y estabilidad operativa. |
| **TC_012** | AC 5: Resiliencia DB | Continuidad del negocio y no pérdida de datos. |
| **TC_013** | AC 6 / DoD: Contract Test | Estabilidad de la interfaz de usuario (Frontend). |