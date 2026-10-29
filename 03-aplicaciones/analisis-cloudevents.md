# analisis-cloudevents.md
# Completa este archivo con tus respuestas al TODO 4 de la Fase 3.

## TODO 4: Análisis de Trazabilidad (CloudEvents)

### 1. El atributo `id`: ¿quién lo genera y para qué sirve?

> Cuando tu aplicación publica un mensaje a través del endpoint de Pub/Sub de Dapr en formato JSON simple, el *sidecar* de Dapr (`daprd`) envuelve automáticamente el contenido dentro de un sobre (*envelope*) CloudEvents v1.0 y genera un **UUID v4 único** para dicho evento

### 2. El atributo `source`: ¿qué valor tiene y de dónde lo toma Dapr?

> _Tu respuesta aquí_

### 3. El atributo `traceparent` y su relación con OpenTelemetry

> _Tu respuesta aquí_

### 4. Valor de negocio para auditoría: atributos más útiles del sobre CloudEvents

> _Tu respuesta aquí_
