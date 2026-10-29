# analisis-abstraccion.md
# Completa este archivo con tus respuestas a los TODOs de la Fase 2.

## 2: Análisis de la Abstracción

### 1. Migración de broker: ¿qué archivos cambiarían?

> Para cambiar el broker de Redis a Apache Kafka se debe modificar en el archivo `pubsub-redis.yaml` el campo `spec.type` a `pubsub.kafka` y se actualizarían las propiedades de metadatos específicas de Kafka. No habría cambios en Python ni en el subscriber ni en el publisher debido a que la API de Pub/Sub de Dapr abstrae por completo la tecnología de mensajería subyacente.

### 2. Inventario de 5 brokers soportados por Dapr

| Broker | `spec.type` en el YAML |
|---|---|
| Redis | `pubsub.redis` |
|Apache Kafka | `pubsub.kafka` |
|RabbitMQ | `pubsub.rabbitmq` |
|AWS SNS/SQS | `pubsub.snssqs` |
|GCP Pub/Sub | `pubsub.gcp.pubsub` |
|Azure Service Bus Topics | `pubsub.azure.servicebus.topics` | 

### 3. Manejo seguro de secretos en producción

> En vez de definir la clave directamente con un valor plano (`value: "xxx"`), se hace referencia al secreto utilizando el mecanismo **secretKeyRef** junto con la sección de autenticación **auth.secretStore**.
