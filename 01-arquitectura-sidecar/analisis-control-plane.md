# analisis-control-plane.md
# Completa este archivo con tus respuestas a los TODOs de la Fase 1.

## 1: Análisis del Plano de Control

### 1. ¿Qué tipo de objeto de Kubernetes es el `dapr-sidecar-injector`?

> dapr-sidecar-injector es un POD. Se requiere la configuración MutatingWebhookConfiguration porque su función principal es **modificar (** **mutar** **) el manifiesto YAML de los Pods 'on the fly'** (de forma dinámica, automática y en memoria) durante la fase de admisión en la API de Kubernetes.


### 2. Réplicas del `dapr-operator` y comparación con Istio

> Actualmente solo hay una réplica del dapr-operator.
Dapr es más liviano que Istio por tres razones principales:

* Operan en capas distintas: Istio intercepta y analiza todo el tráfico de red del clúster (Capas 4 y 7) usando proxies pesados (Envoy). Dapr trabaja a nivel de aplicación, atendiendo solo las llamadas explícitas que tu código hace a sus APIs (HTTP/gRPC).
* Plano de control especializado: Los componentes de Dapr (`operator`, `injector`, `sentry`) solo administran configuraciones y certificados básicos. No necesitan calcular ni sincronizar complejas tablas de enrutamiento de red global.
* Sidecar más ligero: El proceso `daprd` es un ejecutable compacto enfocado en responder a llamadas de API, consumiendo mucha menos memoria y CPU que un proxy de red completo.

### 3. Flujo de inyección del sidecar

> * Envío de la solicitud ( kubectl apply ): Se carga el YAML a Kubernetes con la anotación `dapr.io/enabled: "true"`.
* Kubernetes intercepta: Antes de crear el Pod, la API llama al webhook `dapr-sidecar-injector`.
* Inyección en memoria: El webhook detecta la anotación y le añade automáticamente el contenedor de Dapr ( daprd ) al manifiesto.
* Asignación al nodo: Kubernetes guarda el Pod modificado (ahora con 2 contenedores) y lo asigna a un nodo del clúster.
* Arranque: El nodo inicia tu aplicación y el sidecar de Dapr en paralelo hasta quedar en estado `Running (2/2)`

---

## 2: Exploración de los CRDs de Dapr

### 4. CRDs instalados por Dapr

> * **components.dapr.io**: Configura las conexiones hacia la infraestructura externa (como bases de datos, brokers de mensajes o almacenes de secretos).
* **configurations.dapr.io**: Define reglas globales de comportamiento para el runtime de Dapr, incluyendo tracing/métricas (observabilidad), reglas de mTLS, listas de acceso a APIs y ajustes de actores.
* **resiliencies.dapr.io**: Define políticas de resiliencia (reintentos, timeouts y circuit breakers) aplicables a las llamadas entre servicios o a componentes externos.
* **subscriptions.dapr.io**: Permite definir suscripciones del patrón Pub/Sub de forma puramente declarativa en YAML, vinculando un tópico de mensajería con la ruta HTTP/gRPC de la aplicación sin hardcodearlo en código.
* **httpendpoints.dapr.io**: Configura endpoints HTTP externos para que los microservicios puedan invocar APIs o servicios de terceros fuera del clúster de forma segura mediante Service Invocation

### 5. Relación entre `components.dapr.io` y el concepto de Building Block

> Un **Building Block** es la **API unificada y agnóstica** (de alto nivel) que expone el sidecar de Dapr hacia el código de la aplicación (ejemplo: la API de Estado, Pub/Sub, Secretos o Workflows). Para que un Building Block funcione, necesita apoyarse en uno o más recursos de tipo **components.dapr.io**, los cuales le indican a Dapr con qué infraestructura específica debe interactuar tras bambalinas.
