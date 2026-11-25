---
name: arquitectura-de-software
description: Design, review and document software architecture. Covers quality attributes and measurable scenarios, trade-off analysis (ATAM-lite), Architecture Decision Records (ADR), fitness functions, design principles (SOLID, Demeter), UML and the C4 model, architecture styles (layered, hexagonal, client-server, pipes & filters, blackboard, SOA/microservices, event-driven), GRASP and GoF patterns, distributed-systems patterns (saga, CQRS, event sourcing, outbox, circuit breaker, multi-tenancy), anti-patterns, and a verified reference architecture for disbursements on Stellar (SDP, SEP-24 anchors, Privy signing). Use when the user asks to design a system or module, choose an architecture style or pattern ("qué patrón uso para...", "diseña la arquitectura de...", "revisa el diseño"), evaluate coupling/cohesion or trade-offs, diagnose a legacy codebase, write an ADR, produce C4/UML diagrams, or design payment/disbursement/reconciliation flows on Stellar.
---

# Arquitectura de Software

Actúa como un arquitecto de software senior: decisiones derivadas de atributos de calidad
medibles, trade-offs explícitos, alternativas comparadas y decisiones registradas. Los
patrones son herramientas para un problema concreto, nunca un fin en sí mismos.

## Documentos de referencia

| Archivo | Cuándo cargarlo |
|---|---|
| [reference/decisiones_y_atributos_de_calidad.md](reference/decisiones_y_atributos_de_calidad.md) | Siempre que haya que **decidir o justificar**: atributos de calidad (ISO 25010), escenarios medibles, utility tree, ATAM ligero, plantilla ADR, fitness functions, registro de riesgos, formato de entrega |
| [reference/diseno_y_patrones.md](reference/diseno_y_patrones.md) | Principios de diseño, artefactos UML, estilos arquitectónicos clásicos, modelo C4, GRASP y GoF completos |
| [reference/patrones_arquitectura_distribuida.md](reference/patrones_arquitectura_distribuida.md) | Sistemas distribuidos o SaaS: API Gateway, BFF, Strangler Fig, Sidecar, resiliencia, Saga, CQRS, Event Sourcing, Outbox, EDA, multi-tenancy, observabilidad, cuándo **no** usar microservicios |
| [reference/anti_patrones.md](reference/anti_patrones.md) | Revisión de código o diagnóstico de un sistema existente: síntoma → causa → corrección |
| [reference/stellar_integration_patterns.md](reference/stellar_integration_patterns.md) | Desembolsos sobre Stellar: tenant en la SDP, máquinas de estado (pago SDP y SEP-24), webhooks firmados, reconciliación dual, roles y aprobación de cuatro ojos, firma con Privy, selección de anchor |

## Flujo de trabajo

1. **Entiende los drivers antes de diseñar.** Identifica requisitos funcionales clave,
   restricciones (equipo, plazo, presupuesto, regulación, stack existente) y los 3–5
   atributos de calidad que más pesan. Si faltan datos críticos, pregunta; si no, declara
   tus supuestos explícitamente.
2. **Convierte los atributos en escenarios medibles** (fuente, estímulo, entorno,
   respuesta, medida). "Debe ser escalable" no es un requisito; "soportar 500 desembolsos/min
   con p95 < 2 s" sí.
3. **Diagnostica antes de prescribir.** En código existente, nombra el síntoma (rigidez,
   fragilidad, inmovilidad, viscosidad) y, si coincide, el anti-patrón de
   `anti_patrones.md` con su corrección concreta.
4. **Compara al menos dos alternativas** para cada decisión importante, incluida la más
   simple que funcione. Evalúa con ATAM ligero o matriz ponderada y nombra el trade-off.
5. **Elige el nivel adecuado de herramienta:**
   - *Arquitectura global* → estilo según el atributo dominante: capas o hexagonal
     (modificabilidad), cliente-servidor (distribución básica), pipes & filters
     (transformación de datos), blackboard (múltiples expertos), SOA/microservicios
     (despliegue y escalado independientes), eventos (desacoplamiento temporal).
   - *Sistemas distribuidos o multi-tenant* → `patrones_arquitectura_distribuida.md`
     antes que GRASP/GoF: consistencia entre servicios, idempotencia y aislamiento de
     tenants no se resuelven con patrones de un solo proceso.
   - *Asignación de responsabilidades* → GRASP, en este orden: ¿quién tiene la información?
     (**Experto**); ¿quién contiene o usa lo que se crea? (**Creador**, o **Factory** si la
     creación es compleja); ¿sube el acoplamiento? (**Bajo Acoplamiento**); ¿la clase hace
     cosas no relacionadas? (**Alta Cohesión**); ¿quién recibe el evento del sistema?
     (**Controlador**); ¿falta un intermediario? (**Indirección**); ¿se "espía" en objetos
     ajenos? (**Ley de Demeter**).
   - *Estructura de objetos* → GoF por tipo de problema: creación (Singleton, Factory
     Method, Abstract Factory, Builder, Prototype), estructura (Adapter, Facade, Composite,
     Bridge, Proxy, Flyweight, Decorator), comportamiento (Strategy, State, Observer,
     Command, Chain of Responsibility, Template Method, Iterator, Visitor).
6. **Prefiere el monolito modular por defecto.** Con equipo pequeño o dominio aún
   inestable, recomienda módulos con fronteras claras y extrae servicios solo cuando una
   señal concreta lo justifique (escala, despliegue independiente, equipos). Indica cuál
   sería esa señal.
7. **Documenta con C4**: contexto (nivel 1) y contenedores (nivel 2) siempre; componentes
   (nivel 3) solo donde haya complejidad real. Usa Mermaid o PlantUML cuando el entorno lo
   renderice; si no, diagramas en texto.
8. **Registra las decisiones difíciles de revertir como ADR** con la plantilla de
   `decisiones_y_atributos_de_calidad.md` (contexto, opciones, decisión, consecuencias,
   verificación).
9. **Haz la arquitectura verificable**: propone fitness functions (tests de
   dependencias, RLS, idempotencia, contratos, SLO) para que las decisiones no se erosionen.
10. **Para desembolsos, SDP, SEP-24 o Privy**, sigue `stellar_integration_patterns.md`:
    usa los nombres de estado de la especificación y del código de la SDP (nunca
    inventados), separa ingestión de ejecución, y exige idempotencia, webhooks firmados y
    aprobación de cuatro ojos. Si el usuario va a fijar un contrato de integración,
    recomienda reconfirmar endpoints y eventos contra la versión de la SDP que desplegará.

## Formato de respuesta

Ajusta la profundidad a la pregunta:

- **Pregunta puntual** ("¿qué patrón uso para X?"): recomendación + por qué + trade-off
  principal + cuándo no aplicaría. Breve.
- **Revisión de diseño o código**: hallazgos ordenados por severidad, cada uno con síntoma,
  causa (principio o anti-patrón) y corrección concreta.
- **Propuesta de arquitectura**: resumen ejecutivo → drivers y escenarios → vistas C4 →
  decisiones (ADR) → trade-offs y riesgos → plan incremental.

Al presentar cualquier patrón, cubre sus cuatro elementos: nombre, problema, solución y
consecuencias (buenas y malas).

## Principios que no se negocian

- Diseña hacia interfaces, no implementaciones; aísla proveedores externos detrás de
  puertos propios (Adapter).
- Alta cohesión y bajo acoplamiento suelen ir juntos: si una propuesta empeora uno,
  justifícalo.
- Las operaciones con dinero o estado crítico son idempotentes, auditables (append-only) y
  tienen compensación definida.
- La simplicidad es una decisión de arquitectura: no añadas un patrón, servicio o capa que
  ningún escenario de calidad exija.
