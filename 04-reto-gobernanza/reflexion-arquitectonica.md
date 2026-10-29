# reflexion-arquitectonica.md
# Completa este archivo con tus respuestas al Reto 3 de la Fase 4.

## Reto 3: Reflexión Arquitectónica

### 1. Sin Dapr: ¿dónde vive el código de TTL y consumer group?

> * **TTL:** Kafka no soporta expiración individual por mensaje de forma nativa a nivel de broker (solo retención por tópico a nivel de infraestructura `retention.ms`). En una arquitectura sin Dapr, el código para verificar si un mensaje ha expirado tendría que escribirse manualmente dentro del **código del suscriptor**, comprobando la estampa de tiempo (*timestamp*) de cada mensaje antes de procesarlo.
* **Consumer Group:** Se configuraría explícitamente dentro del código de inicialización del SDK de Kafka (por ejemplo, en el diccionario de parámetros del cliente `group.id = "order-group"`).

### 2. El patrón sidecar como frontera de responsabilidad

> * **Equipo de Producto / Desarrollo (Regla de negocio):**
* Define la lógica de negocio y los Acuerdos de Nivel de Servicio (SLA). Por ejemplo: *"Una orden no procesada en 2 minutos se vuelve inválida"*.
* Puede especificar esta regla definiendo el campo `ttlInSeconds: "120"` dentro de los metadatos al publicar la orden o en la especificación del componente.
* **Equipo de Plataforma / DevOps (Infraestructura y GitOps):**
* Mantiene y despliega los manifiestos YAML de los componentes (`Component` CRDs) mediante tuberías de CI/CD (GitOps).
* Configura las credenciales, conexiones, almacenamiento seguro, resiliencia y el afinamiento del broker

### 3. Limitaciones de la abstracción: ¿migración 100% transparente?

> Aunque Dapr permite cambiar de Redis a Kafka modificando únicamente la especificación del componente YAML sin tocar el código fuente, **la migración no es 100% transparente en el comportamiento operativo del sistema** debido a diferencias en los motores subyacentes:

1. **Diferencias en el soporte de TTL y Consumidores No-Dapr:**
  * Dapr aplica el filtrado de mensajes expirados directamente en su *runtime* antes de entregar el evento a la aplicación.
  * Sin embargo, si existen **suscriptores externos que no usan Dapr** leyendo directamente del broker:
    * En brokers con soporte nativo de TTL (como Azure Service Bus), el propio broker descarta o mueve el mensaje expirado a una cola de mensajes no entregados (*Dead Letter Queue*).
    * En brokers como Redis Streams o Kafka, los mensajes expirados permanecen en el almacenamiento y los consumidores externos No-Dapr los recibirán y procesarán a menos que implementen lógica manual para verificar la propiedad de expiración en el sobre de CloudEvents.
2. **Modelos de concurrencia y particionamiento (Limitación concreta):**
  * **Redis Streams** escala el consumo agrupando suscriptores mediante `consumerID` y listas pendientes.
  * **Apache Kafka** acota la concurrencia estrictamente al número de **particiones** asignadas a un tópico. Si escalas tu aplicación suscriptora a 10 réplicas pero el tópico de Kafka solo fue creado con 3 particiones, 7 réplicas quedarán inactivas (*idle*). Esta es una regla estructural del motor de Kafka que la abstracción de Dapr no puede saltarse.
