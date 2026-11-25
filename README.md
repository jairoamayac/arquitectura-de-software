# Arquitectura de software

Skill de [Claude Code](https://claude.com/claude-code) para diseñar, revisar y documentar
arquitectura de software con el método de un arquitecto senior: drivers y atributos de
calidad medibles, alternativas comparadas, trade-offs explícitos, decisiones registradas
como ADR y reglas verificables en CI.

Incluye una arquitectura de referencia **verificada contra el código fuente** para
desembolsos sobre Stellar: **Stellar Disbursement Platform (SDP)**, anchors **SEP-24** y
firma con **Privy**.

## ¿Qué hace esta skill?

Claude Code la activa automáticamente cuando el usuario:

- Pide diseñar o revisar la arquitectura de un sistema o módulo.
- Necesita elegir entre estilos o patrones y justificar la decisión.
- Pide un ADR, un análisis de trade-offs o escenarios de atributos de calidad.
- Pide diagramas C4 o UML (clases, componentes, despliegue).
- Quiere diagnosticar un código legacy (síntomas de mal diseño, anti-patrones).
- Diseña flujos de pagos, desembolsos o reconciliación con SDP o anchors SEP.

## Contenido

| Archivo | Cubre |
|---|---|
| [`SKILL.md`](SKILL.md) | Flujo de trabajo: drivers → escenarios medibles → diagnóstico → alternativas y trade-offs → C4 → ADR → fitness functions; formato de respuesta según el tipo de pregunta |
| [`reference/decisiones_y_atributos_de_calidad.md`](reference/decisiones_y_atributos_de_calidad.md) | Atributos de calidad (ISO/IEC 25010) y tácticas, escenarios SEI, utility tree, ATAM ligero, matriz de decisión, plantilla ADR, fitness functions, registro de riesgos, formato de propuesta |
| [`reference/diseno_y_patrones.md`](reference/diseno_y_patrones.md) | Principios de diseño, UML, estilos arquitectónicos (capas, hexagonal, blackboard, pipes & filters, cliente-servidor, SOA/microservicios), DDD estratégico, modelo C4, GRASP y GoF completos |
| [`reference/patrones_arquitectura_distribuida.md`](reference/patrones_arquitectura_distribuida.md) | API Gateway, BFF, Strangler Fig, Sidecar, Circuit Breaker, Saga, CQRS, Event Sourcing, Outbox, EDA, multi-tenancy (BD / esquema / fila + RLS), observabilidad, y cuándo **no** usar microservicios |
| [`reference/anti_patrones.md`](reference/anti_patrones.md) | Síntoma → causa → corrección: God Object, Anemic Domain Model, Feature Envy, Shotgun Surgery, Big Ball of Mud, Golden Hammer, Distributed Monolith, Chatty Services, Dual Write sin Outbox, entre otros |
| [`reference/stellar_integration_patterns.md`](reference/stellar_integration_patterns.md) | Vista C4 de la integración, resolución de tenant en la SDP, máquinas de estado (desembolso, pago, SEP-24), webhooks con outbox y HMAC, reconciliación dual, roles y aprobación de cuatro ojos, firma con Privy (raw sign), selección de anchor, checklist de revisión |

## Instalación

```bash
git clone https://github.com/jairoamayac/arquitectura-de-software.git \
  ~/.claude/skills/arquitectura-de-software
```

Claude Code detecta la skill (carpeta con un `SKILL.md` válido) al iniciar una nueva
sesión. Para actualizarla:

```bash
git -C ~/.claude/skills/arquitectura-de-software pull
```

## Fundamentos y referencias

- Material de **"Fundamentos de Ingeniería de Software — Diseño"** y **"— Patrones"**,
  Pontificia Universidad Javeriana (Ing. Orlando Vasquez).
- Bass, Clements, Kazman — *Software Architecture in Practice* (atributos de calidad,
  escenarios, ATAM).
- Gamma, Helm, Johnson, Vlissides — *Design Patterns* (GoF).
- Larman — *Applying UML and Patterns* (GRASP).
- Evans — *Domain-Driven Design*.
- Richardson — *Microservices Patterns* (Saga, Outbox, CQRS).
- Ford, Parsons, Kua — *Building Evolutionary Architectures* (fitness functions).
- Nygard — *Documenting Architecture Decisions* (ADR).
- Brown — *The C4 Model for Visualising Software Architecture*.
- Stellar Development Foundation — código de `stellar-disbursement-platform-backend`,
  especificaciones SEP-10/12/24/38 y documentación de la SDP.

## Licencia

Uso educativo y profesional. El material académico base conserva la autoría de su fuente.
