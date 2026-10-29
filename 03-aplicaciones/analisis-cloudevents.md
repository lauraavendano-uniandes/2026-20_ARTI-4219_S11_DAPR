# analisis-cloudevents.md
# Completa este archivo con tus respuestas al TODO 4 de la Fase 3.

## TODO 4: Análisis de Trazabilidad (CloudEvents)

### 1. El atributo `id`: ¿quién lo genera y para qué sirve?

> Cuando la aplicación publica un mensaje a través del endpoint de Pub/Sub de Dapr en formato JSON simple, el *sidecar* de Dapr (`daprd`) envuelve automáticamente el contenido dentro de un sobre (*envelope*) CloudEvents v1.0 y genera un **UUID v4 único** para dicho evento

### 2. El atributo `source`: ¿qué valor tiene y de dónde lo toma Dapr?

> El valor del campo `source` identifica el origen contextual o emisor del evento en la red de microservicios.
Dapr toma este valor directamente del parámetro o anotación **`app-id`** (`dapr.io/app-id` en los manifiestos de Kubernetes o el flag `--app-id` de la CLI/daprd) correspondiente a la aplicación que publicó el mensaje. 

### 3. El atributo `traceparent` y su relación con OpenTelemetry

> Implementa la especificación estándar **W3C Trace Context**.
**OpenTelemetry (OTel)** adopta W3C Trace Context como su formato nativo de propagación contextual de trazado distribuido. 

Cuando el sidecar de Dapr envuelve la carga útil en el sobre CloudEvents, inyecta la cabecera `traceparent`. De este modo, la información de rastreo (`trace-id` y `span-id`) viaja empaquetada a través del intermediario de mensajería. Al recibir el evento, el sidecar del suscriptor extrae el `traceparent` y continúa la traza. Esto permite que colectores de observabilidad compatibles con OpenTelemetry (como Jaeger, Zipkin o Azure Monitor) puedan correlacionar y reconstruir la traza completa de extremo a extremo a través de múltiples servicios asíncronos.

### 4. Valor de negocio para auditoría: atributos más útiles del sobre CloudEvents

> Si un equipo de auditoría necesita rastrear y reconstruir todos los eventos procesados en el sistema en las últimas 24 horas, los atributos más relevantes del sobre CloudEvents son:

1. **`time`**: Proporciona la marca de tiempo precisa en estándar **RFC3339** (ej. `2026-10-29T15:00:00Z`) indicando cuándo fue emitido el evento. Es vital para delimitar con exactitud la ventana temporal de las 24 horas y ordenar la cronología del flujo.
2. **`id`**: Permite realizar auditorías de recuento exacto sin duplicados, rastrear transacciones individuales y contrastar logs entre emisores y receptores.
3. **`source`**: Identifica inequívocamente qué servicio o sistema generó cada evento, asegurando la no repudiación y la trazabilidad de origen.
4. **`type`**: Describe la naturaleza o clase del evento de negocio (ej. `order.created`, `payment.processed`), permitiendo a los auditores filtrar y analizar únicamente los tipos de transacciones de interés.
5. **`traceparent`**: Permite vincular cada evento dentro del registro de auditoría directamente con las trazas de ejecución en la infraestructura de observabilidad.