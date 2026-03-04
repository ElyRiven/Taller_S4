# SUITE DE CASOS DE PRUEBA: MIGRACIÓN CLEAN ARCHITECTURE (US-003)
**Componente:** Consumer-Worker  
**Enfoque:** Desacoplamiento, resiliencia e integridad de datos.

---

## TC_009: Desacoplamiento de Paginación (Capa Application/Domain)
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_009 |
| **Prioridad** | Alta |
| **Propósito** | Verificar que el núcleo no dependa de Spring Data (Inversión de Dependencias). |

### Pasos de Ejecución
1. Inspeccionar imports en `PagedKudoResponse` (Domain/Application).
2. Verificar ausencia de `org.springframework.data.domain.Page`.
3. Ejecutar `./gradlew dependencies` y validar aislamiento del núcleo.

**Resultado Esperado:** Compilación exitosa. El núcleo es independiente del framework de persistencia.

---

## TC_010: Flujo de Persistencia de Kudos vía RabbitMQ
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_010 |
| **Prioridad** | Alta |
| **Propósito** | Validar delegación correcta del listener al puerto de salida y manejo de idempotencia. |



### Pasos de Ejecución
1. Publicar mensaje JSON válido en `kudos.queue`.
2. Consultar PostgreSQL para verificar persistencia.
3. Forzar un **redelivery** del mismo mensaje y verificar que no haya duplicados.

**Resultado Esperado:** Procesamiento exitoso e idempotente. Un solo registro en BD por mensaje.

---

## TC_011: Manejo de Errores y Enrutamiento a DLQ
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_011 |
| **Prioridad** | Media |
| **Propósito** | Confirmar que mensajes malformados se aislan en la Dead Letter Queue. |

### Pasos de Ejecución
1. Enviar JSON inválido a la cola principal.
2. Confirmar en logs el lanzamiento de `InvalidKudoException`.
3. Verificar que el mensaje se mueva a `kudos.dlq`.
4. Consultar el conteo de mensajes en la DLQ y validar que coincida con la cantidad de mensajes fallidos enviados.
5. Verificar que existan métricas o alertas asociadas a la DLQ (ej. contador de mensajes fallidos, logs con nivel ERROR/WARN con metadata del mensaje rechazado).
6. Enviar un mensaje válido inmediatamente después para confirmar continuidad del servicio.

**Resultado Esperado:** El mensaje inválido se aisla en la DLQ con trazabilidad (conteo y métricas/alertas), y el sistema sigue procesando mensajes válidos sin detenerse.

---

## TC_012: Resiliencia ante Indisponibilidad de Base de Datos
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_012 |
| **Prioridad** | Alta |
| **Propósito** | Asegurar que no se pierdan Kudos si PostgreSQL falla (Retry & Backoff). |

### Pasos de Ejecución
1. Detener el contenedor de PostgreSQL.
2. Enviar mensajes a la cola.
3. Verificar en logs la política de reintentos (backoff).
4. Iniciar PostgreSQL y observar el procesamiento automático.

**Resultado Esperado:** Mensajes permanecen en RabbitMQ durante la caída. Al restaurar la BD, se procesan automáticamente sin pérdida.

---

## TC_013: Validación de Contrato de API (Contract Testing)
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_013 |
| **Prioridad** | Media |
| **Propósito** | Asegurar compatibilidad del JSON con el Frontend (Contract-First), incluyendo versionado y backward compatibility. |

### Pasos de Ejecución
1. Ejecutar tests de contrato (`src/test/resources/contracts/`).
2. Alterar un campo en el DTO y confirmar que el test detecta la ruptura del contrato.
3. Verificar que el contrato incluya un esquema de versionado (ej. header `API-Version`, path `/v1/`, o campo en el JSON).
4. Validar backward compatibility: introducir un campo nuevo opcional en el contrato y confirmar que los consumidores existentes (Frontend) sigan funcionando sin modificaciones.

**Resultado Esperado:** El test garantiza que el JSON de salida no rompa la integración con el Frontend, el contrato está versionado y es compatible hacia atrás con consumidores existentes.

---

## 📊 MATRIZ DE TRAZABILIDAD (RTM)
| ID Caso | Requisito (US-003) | Objetivo de Negocio |
| :--- | :--- | :--- |
| TC_009 | AC 1: Desacoplamiento | Mantenibilidad técnica |
| TC_010 | AC 2: Consumo exitoso | Persistencia confiable |
| TC_011 | AC 4: Manejo de inválidos | Estabilidad operativa |
| TC_012 | AC 5: Resiliencia DB | No pérdida de datos |
| TC_013 | AC 6: Contract Test | Estabilidad del Frontend |