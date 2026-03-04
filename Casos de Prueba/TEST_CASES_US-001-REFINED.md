# SUITE DE CASOS DE PRUEBA: SEGURIDAD Y OPTIMIZACIÓN (US-001)
**Proyecto:** SofkianOS-MVP  
**Enfoque:** Seguridad, eficiencia de imágenes y control de calidad en CI/CD.

---

## TC_001: Validación de Arquitectura Multi-stage en Componentes de Kudos
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_001 |
| **Prioridad** | Alta |
| **Propósito** | Verificar que los Dockerfiles utilicen construcción multi-stage real, eliminando dependencias de compilación del artefacto final. |

### Pasos de Ejecución
1. Abrir el Dockerfile del componente a validar.
2. Verificar la existencia de múltiples etapas (`AS build`, `AS runtime`).
3. Confirmar el uso de `COPY --from` únicamente para artefactos finales (`dist/`, `.jar`).
4. Construir la imagen Docker.
5. Inspeccionar la imagen final y validar la ausencia de SDKs, JDKs, Node o herramientas de build.
6. Comparar el tamaño de la imagen con una versión sin multi-stage (si existe).

**Resultado Esperado:** La imagen final es ligera, no contiene herramientas de compilación y es apta para despliegues Rolling Update.

---

## TC_002: Ejecución de Contenedores con Usuario No-Root (appuser)
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_002 |
| **Prioridad** | Alta |
| **Propósito** | Garantizar que los servicios se ejecuten sin privilegios de administrador. |

### Pasos de Ejecución
1. Asegurar que el contenedor esté en ejecución.
2. Ejecutar `docker exec [ID_CONTENEDOR] whoami`.
3. Intentar crear un archivo en `/root/`.
4. Ejecutar una operación funcional del servicio (ej. request HTTP).

**Resultado Esperado:** El proceso corre bajo `appuser`, el acceso a `/root/` es denegado y el servicio funciona correctamente.

---

## TC_003: Bloqueo de PR por Vulnerabilidades CRÍTICAS en CI/CD
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_003 |
| **Prioridad** | Alta |
| **Propósito** | Bloquear despliegues con vulnerabilidades críticas que comprometan datos sensibles. |

### Pasos de Ejecución
1. Introducir una dependencia vulnerable **CRITICAL** controlada.
2. Abrir un Pull Request hacia `main`.
3. Monitorear el paso de análisis de seguridad (Trivy) en el CI.

**Resultado Esperado:** El pipeline falla, se reporta la vulnerabilidad y el PR queda bloqueado para merge.

---

## TC_004: Continuidad de Pipeline con Vulnerabilidades LOW / MEDIUM
| Campo | Detalle |
| :--- | :--- |
| **ID** | TC_004 |
| **Prioridad** | Media |
| **Propósito** | Mantener la agilidad del MVP permitiendo el avance ante riesgos de bajo impacto. |

### Pasos de Ejecución
1. Introducir dependencias con vulnerabilidades **LOW** o **MEDIUM**.
2. Ejecutar el pipeline CI y revisar logs del escáner.
3. Verificar que las vulnerabilidades detectadas queden registradas con trazabilidad completa (severidad, CVE, paquete afectado) en los logs del pipeline.
4. Confirmar que el reporte de vulnerabilidades sea accesible como artefacto del pipeline o en el resumen del check de GitHub.

**Resultado Esperado:** El pipeline finaliza exitosamente. Las vulnerabilidades se reportan como advertencia con trazabilidad completa (severidad, CVE, componente) sin bloquear el flujo, y quedan disponibles para auditoría.

---

## 📊 MATRIZ DE TRAZABILIDAD – US-001
| ID Caso | Requisito | Objetivo de Negocio |
| :--- | :--- | :--- |
| TC_001 | Multi-stage real | Optimización de recursos |
| TC_002 | Usuario no-root | Seguridad operativa |
| TC_003 | Bloqueo CRITICAL | Protección de datos |
| TC_004 | Continuidad LOW/MED | Velocidad del MVP |