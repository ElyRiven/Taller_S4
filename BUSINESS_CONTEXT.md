# Plantilla para Diligenciar Contexto de Negocio

## 1. Descripción del Proyecto
* **Nombre del Proyecto:** SofkianOS-MVP
* **Objetivo del Proyecto:**  Transformar la identidad y los valores de los colaboradores ("Sofkianos") en un sistema de reconocimiento tangible llamado **Kudos**. El proyecto busca fomentar una cultura de agradecimiento y recompensas que fortalezca los vínculos en equipos distribuidos geográficamente, permitiendo que el desempeño y la actitud positiva sean visibilizados y celebrados.

---

## 2. Flujos Críticos del Negocio
* **Principales Flujos de Trabajo:**
  * **Otorgamiento de Kudos:** Proceso mediante el cual un usuario reconoce el trabajo de un compañero enviando Kudos asociados a un valor cultural.
  * **Visualización de todos los Kudos:** Flujo para consultar todos los kudos almacenados en una base de datos y filtrarlos por usuario que envió el kudo, usuario que recibió el kudo, descripción y rango de fechas.
* **Módulos o Funcionalidades Críticas:** 
  * **Frontend (React v19.2):** Aplicación de frontend vital para el acceso al sistema, ya que contiene la landing page, el acceso al formulario de envío de kudos y el panel de consulta de kudos existentes.
  * **Producer API (Spring v3.4.5, Java 17):** Microservicio creado con Spring para la generación de mensajes que se almacenan el RabbitMQ para conexión asíncrona.
  * **Consumer Worker (Spring v3.4.5, Java 17):** Microservicio creado con Spring para la obtención de los mensajes almacenados por RabbitMQ y almacenar su información en una DB Postgres. Además este microservicio expone una API para la consulta de Kudos existentes.

---

## 3. Reglas de Negocio y Restricciones
* **Reglas de Negocio Relevantes:** * *
  *  Todos los kudos deben ser enviados y recibidos por usuarios válidos en el sistema.
  *  Los kudos deben contener una categoría válida para ser enviados.
* **Regulaciones o Normativas:**
  * **Protección de Datos (GDPR/Ley 1581):** Tratamiento de la información personal de los empleados y su historial de desempeño.
  * **Políticas Internas de Sofka:** Alineación con los estándares éticos y de comunicación de la compañía.

---

## 4. Perfiles de Usuario y Roles
* **Perfiles o Roles de Usuario en el Sistema:** * *
  * Usuario genérico que tiene acceso al sistema sin necesidad de autenticación o identificación previa. 
* **Permisos y Limitaciones de Cada Perfil:**
  * El Usuario genérico tiene la capacidad de acceder al envío de kudos y consulta de kudos existentes.

---

## 5. Condiciones del Entorno Técnico
* **Plataformas Soportadas:** 
  * **Web:** Interfaz principal para la gestión y visualización detallada.
  * **Mobile Friendly:** Diseño adaptable para facilitar el reconocimiento inmediato desde dispositivos móviles.
* **Tecnologías o Integraciones Clave:** 
  * **RabbitMQ**: Broker de mensajería que almacena en una cola los mensajes enviados por el microservicio Producer API.
  * **PostgreSQL**: Base de datos relacional que almacena la información enviada mediante el microservicio Consumer Worker.

---

## 6. Casos Especiales o Excepciones (Opcional)
* **Escenarios Alternos o Excepciones:** 
  * El sistema no está diseñado para permitir la eliminación ni actualización de los kudos; ningún registro creado en la base de datos se puede eliminar ni modificar.
  * El sistema no integra ningún método de autenticación. El usuario puede acceder a todo el contenido web del sistema sin restricción.