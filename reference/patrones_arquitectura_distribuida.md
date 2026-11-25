# Patrones de Arquitectura Distribuida y Microservicios

> Complemento a los patrones clásicos (GRASP/GoF/estilos arquitectónicos). Cubre los patrones que
> dominan el diseño de sistemas distribuidos, SaaS multi-tenant y arquitecturas modernas
> basadas en eventos — necesarios cuando la arquitectura objetivo va más allá de un monolito
> en capas.

---

## 1. Patrones de comunicación e integración

### API Gateway
Un único punto de entrada que enruta solicitudes de clientes a los microservicios correspondientes, centralizando auth, rate-limiting, logging y composición de respuestas.

- **Problema**: clientes que deben conocer y llamar a N servicios directamente → acoplamiento alto, lógica de cross-cutting duplicada.
- **Solución**: un gateway enruta, agrega, y aplica políticas transversales (autenticación, throttling) antes de llegar a los servicios internos.
- **Cuándo NO usarlo**: sistemas pequeños con pocos servicios — añade un punto de fallo y latencia innecesaria.

### Backend for Frontend (BFF)
Variante del API Gateway: un backend dedicado por tipo de cliente (web, mobile, partner API), en lugar de una API genérica para todos.

- **Problema**: una sola API "talla única" obliga a sobre-fetch o bajo-fetch de datos según el cliente.
- **Solución**: cada experiencia de cliente tiene su propio BFF que compone llamadas a los servicios backend según las necesidades específicas de esa UI.

### Strangler Fig (Migración incremental)
Reemplaza gradualmente un sistema legacy enrutando funcionalidad nueva o migrada a través de una fachada, hasta "ahogar" el sistema viejo.

- **Problema**: reescribir un monolito completo de una vez es riesgoso.
- **Solución**: introducir una capa de fachada/proxy que enruta tráfico al sistema legacy o al nuevo servicio según la funcionalidad ya migrada. Se migra módulo por módulo.

### Sidecar
Un proceso auxiliar que se despliega junto a un servicio (mismo pod/host) para manejar funcionalidad transversal (logging, proxy de red, métricas) sin acoplarla al código del servicio.

- **Ejemplo típico**: Service Mesh (Istio/Linkerd) usa sidecars (Envoy) para manejar enrutamiento, retries y mTLS sin tocar el código de negocio.

---

## 2. Patrones de resiliencia

### Circuit Breaker
Evita llamadas repetidas a un servicio que está fallando, "abriendo el circuito" tras N fallos consecutivos y devolviendo una respuesta de fallback inmediata en vez de esperar timeouts.

- **Estados**: Cerrado (llamadas normales) → Abierto (falla rápido, no llama al servicio) → Semi-abierto (prueba si el servicio se recuperó).
- **Problema que resuelve**: fallos en cascada — un servicio lento/caído puede agotar threads/conexiones de sus clientes.

### Retry con Backoff exponencial
Reintenta una operación fallida con esperas crecientes entre intentos (y jitter aleatorio) para evitar sincronizar reintentos de múltiples clientes ("thundering herd").

### Bulkhead
Aísla recursos (pools de conexión, threads) por dependencia, para que el agotamiento de recursos al llamar a un servicio no afecte las llamadas a otros servicios.

### Timeout + Fallback
Toda llamada a un servicio externo debe tener un timeout explícito y una respuesta de fallback definida (caché, valor por defecto, degradación elegante) — nunca esperar indefinidamente.

---

## 3. Patrones de datos en sistemas distribuidos

### Database per Service
Cada microservicio posee su propia base de datos; otros servicios no acceden directamente a ella, solo vía la API del servicio dueño.

- **Beneficio**: bajo acoplamiento, cada servicio evoluciona su esquema libremente.
- **Costo**: ya no hay transacciones ACID entre servicios — se necesitan los patrones de consistencia siguientes.

### Saga
Coordina una secuencia de transacciones locales a través de múltiples servicios, con pasos de compensación si algo falla a mitad de camino — sustituye a las transacciones distribuidas ACID.

- **Saga Orquestada**: un coordinador central dirige cada paso y dispara compensaciones en orden inverso si falla un paso.
- **Saga Coreografiada**: cada servicio reacciona a eventos de los demás sin un coordinador central (más desacoplado, más difícil de trazar).
- **Ejemplo**: reserva de vuelo + hotel + auto — si falla el hotel, se compensa cancelando el vuelo ya reservado.

### CQRS (Command Query Responsibility Segregation)
Separa el modelo de escritura (comandos, que mutan estado) del modelo de lectura (queries, optimizado para consulta), pudiendo incluso usar almacenamientos distintos para cada uno.

- **Problema que resuelve**: un mismo modelo de datos optimizado para escritura normalizada rara vez es óptimo para lecturas complejas (reportes, dashboards).
- **Cuándo NO usarlo**: CRUD simple sin necesidades de lectura especializadas — añade complejidad innecesaria.

### Event Sourcing
En lugar de almacenar el estado actual, se almacena la secuencia completa de eventos que llevaron a ese estado; el estado actual se reconstruye reproduciendo los eventos (o desde un snapshot).

- **Beneficio**: auditoría completa e inmutable por diseño — natural para dominios donde la trazabilidad es un requisito (ej. movimientos financieros, desembolsos).
- **Suele combinarse con CQRS**: los eventos alimentan el modelo de escritura; vistas materializadas alimentan el modelo de lectura.

### Outbox Pattern
Garantiza que escribir en la base de datos y publicar un evento ocurran de forma atómica: el evento se guarda en una tabla "outbox" dentro de la misma transacción de BD, y un proceso aparte (poller o CDC) lo publica al bus de mensajes después.

- **Problema que resuelve**: el "dual write problem" — escribir en BD y publicar a un message broker en dos pasos separados puede dejar al sistema inconsistente si uno falla y el otro no.

---

## 4. Patrones de mensajería y eventos

### Event-Driven Architecture (EDA)
Los servicios se comunican mediante eventos publicados a un bus/broker (Kafka, SNS/SQS, RabbitMQ) en lugar de llamadas síncronas directas.

- **Pub/Sub**: un publicador emite eventos sin saber quién los consume; múltiples suscriptores reaccionan de forma independiente.
- **Event Notification** vs. **Event-Carried State Transfer**: el evento puede ser solo una notificación (el consumidor debe pedir el detalle) o llevar el estado completo necesario (evita una llamada adicional, pero acopla el contrato del evento).

### Idempotent Consumer
Un consumidor de eventos/mensajes debe poder procesar el mismo mensaje más de una vez (por reintentos, redelivery) sin efectos secundarios duplicados — típicamente usando una clave de idempotencia y un registro de mensajes ya procesados.

- **Crítico en sistemas de pagos/desembolsos**: un reintento de entrega de un evento no puede generar un doble pago.

---

## 5. Patrones de multi-tenancy (SaaS)

### Modelos de aislamiento de datos

| Modelo | Descripción | Trade-off |
|---|---|---|
| **Base de datos por tenant** | Cada tenant tiene su propia BD física | Máximo aislamiento y seguridad; mayor costo operativo y de infraestructura |
| **Esquema por tenant** | Una BD, un esquema (schema) por tenant | Aislamiento medio; más simple de operar que BD separada |
| **Fila por tenant (BD compartida + `tenant_id`)** | Una sola BD/esquema, todas las tablas incluyen `tenant_id`, reforzado con Row Level Security (RLS) | Menor costo operativo; requiere disciplina estricta — **toda query debe filtrar por tenant**, reforzada en dos capas: código + RLS de BD |

### Tenant Context Propagation
El identificador de tenant debe propagarse explícitamente en cada capa: request HTTP → contexto de aplicación → query a BD → llamadas a servicios externos. Es un error común olvidar propagarlo en llamadas asíncronas (colas, workers) o a servicios de terceros.

### Noisy Neighbor
Riesgo del modelo de BD/recursos compartidos: un tenant con uso intensivo puede degradar el rendimiento de los demás. Se mitiga con rate-limiting por tenant, quotas, y particionamiento de colas/recursos críticos por tenant.

---

## 6. Patrones de observabilidad

### Health Check API
Cada servicio expone un endpoint (`/health`, `/ready`) que reporta su estado y el de sus dependencias críticas, usado por orquestadores (Kubernetes) y balanceadores para decidir enrutamiento.

### Distributed Tracing
Cada request recibe un `trace_id` que se propaga a través de todos los servicios que participan en su procesamiento, permitiendo reconstruir el flujo completo de una operación distribuida (ej. OpenTelemetry).

### Log Aggregation
Los logs de todos los servicios se centralizan (ej. ELK, Loki) con campos estructurados consistentes (incluyendo `tenant_id` y `trace_id`) para poder correlacionar eventos entre servicios.

---

## 7. Cuándo NO usar microservicios / EDA

No todo sistema necesita esta complejidad. Señales de que un monolito modular (bien diseñado con los principios GRASP/SOLID del módulo base) es la opción correcta:

- El equipo es pequeño y un solo despliegue es suficiente para el volumen de cambios.
- No hay necesidad real de escalar componentes de forma independiente.
- La complejidad operativa (orquestación, observabilidad distribuida, consistencia eventual) supera el valor que aporta la separación.

> Regla práctica: empieza con un monolito modular bien diseñado (paquetes/capas claras, bajo acoplamiento entre módulos) y extrae microservicios solo cuando un cuello de botella concreto de escalabilidad, despliegue independiente, o equipo lo justifique — no por moda.
