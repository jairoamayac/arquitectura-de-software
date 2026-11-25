# Decisiones Arquitectónicas y Atributos de Calidad

> La arquitectura es el conjunto de decisiones difíciles de revertir. Este documento cubre
> cómo **derivarlas** de los atributos de calidad, cómo **evaluar** sus trade-offs y cómo
> **registrarlas** para que sobrevivan al equipo que las tomó.

---

## 1. Atributos de calidad (ISO/IEC 25010)

Los requisitos funcionales dicen *qué* hace el sistema; los atributos de calidad deciden
la *forma* de la arquitectura. Los más determinantes:

| Atributo | Pregunta que lo revela | Tácticas típicas |
|---|---|---|
| **Rendimiento** | ¿Cuánta latencia/throughput se tolera bajo carga pico? | Caché, colas, CQRS, particionamiento, procesamiento asíncrono |
| **Disponibilidad** | ¿Cuánto tiempo caído se tolera? ¿Qué pasa si cae una dependencia? | Redundancia, health checks, Circuit Breaker, degradación elegante |
| **Escalabilidad** | ¿Qué crece: usuarios, datos, tenants? ¿Linealmente? | Servicios sin estado, sharding, autoescalado |
| **Seguridad** | ¿Qué activo se protege y de quién? | Mínimo privilegio, defensa en profundidad, cifrado, aislamiento de tenants, auditoría |
| **Modificabilidad** | ¿Qué cambia con más frecuencia? ¿Cuánto cuesta cambiarlo? | Bajo acoplamiento, interfaces estables, puertos y adaptadores |
| **Integridad / consistencia** | ¿Qué estado nunca puede quedar a medias? | Transacciones locales, Saga, Outbox, idempotencia |
| **Auditabilidad / trazabilidad** | ¿Hay que reconstruir quién hizo qué y cuándo? | Event sourcing, logs inmutables, append-only |
| **Operabilidad / observabilidad** | ¿Cómo sabemos que algo va mal antes que el usuario? | Tracing distribuido, métricas RED/USE, alertas por SLO |
| **Costo** | ¿Cuál es el presupuesto de infraestructura y de equipo? | Monolito modular, serverless, servicios gestionados |

> Los atributos **compiten**: más consistencia suele costar disponibilidad o latencia;
> más modularidad física (microservicios) cuesta operabilidad. El trabajo del arquitecto
> es hacer explícito el trade-off, no esconderlo.

---

## 2. Escenarios de atributos de calidad

Un atributo sin escenario medible es una opinión. Formato (SEI):

| Parte | Pregunta | Ejemplo |
|---|---|---|
| **Fuente** | ¿Quién o qué provoca el estímulo? | El TSS |
| **Estímulo** | ¿Qué ocurre? | Reintenta un pago por timeout de red |
| **Artefacto** | ¿Qué parte del sistema? | Servicio de desembolsos |
| **Entorno** | ¿En qué condición? | Operación normal, carga pico |
| **Respuesta** | ¿Qué debe hacer el sistema? | Procesar el pago una sola vez |
| **Medida** | ¿Cómo se verifica? | 0 pagos duplicados en 10.000 reintentos simulados |

Prioriza los escenarios en un **utility tree**: cada uno con (importancia de negocio,
dificultad técnica) en escala Alta/Media/Baja. Los (A, A) definen la arquitectura.

---

## 3. Evaluación de trade-offs (ATAM ligero)

Para cada decisión candidata:

1. **Enfoques**: lista al menos dos alternativas reales (incluida "no hacer nada" o "lo
   más simple que funcione").
2. **Escenarios afectados**: qué escenarios del utility tree mejora o empeora cada una.
3. **Puntos de sensibilidad**: parámetros de los que depende el resultado (tamaño de
   lote, timeout, número de tenants).
4. **Trade-off points**: decisiones que mejoran un atributo a costa de otro.
5. **Riesgos y no-riesgos**: qué podría salir mal y qué está razonablemente cubierto.

Matriz de decisión ponderada (útil cuando hay más de dos opciones):

| Criterio (peso) | Opción A | Opción B | Opción C |
|---|---|---|---|
| Consistencia (5) | 4 → 20 | 2 → 10 | 5 → 25 |
| Costo operativo (3) | 5 → 15 | 3 → 9 | 2 → 6 |
| Time-to-market (4) | 4 → 16 | 5 → 20 | 2 → 8 |
| **Total** | **51** | 39 | 39 |

> La matriz no decide sola: sirve para hacer visibles los pesos y discutirlos.

---

## 4. Architecture Decision Records (ADR)

Toda decisión difícil de revertir se registra como ADR en el repositorio
(`docs/adr/NNNN-titulo-en-kebab-case.md`). Los ADR son **inmutables**: si una decisión
cambia, se escribe un ADR nuevo que **reemplaza** al anterior.

### Plantilla (formato Nygard extendido)

```markdown
# ADR-0007: Outbox transaccional para publicar eventos de desembolso

- **Estado**: Propuesto | Aceptado | Reemplazado por ADR-00XX | Obsoleto
- **Fecha**: 2026-11-24
- **Decisores**: <roles, no solo nombres>

## Contexto
Qué fuerza obliga a decidir: requisito, atributo de calidad, restricción, incidente.
Incluye los escenarios de calidad afectados.

## Opciones consideradas
1. Publicar al broker después del commit (dual write).
2. Outbox transaccional + relay.
3. CDC sobre el WAL de PostgreSQL.

## Decisión
Opción 2. Una frase clara en voz activa: "Usaremos…".

## Consecuencias
- Positivas: atomicidad entre estado y evento; reintentos seguros.
- Negativas: latencia adicional (intervalo del relay); tabla que hay que purgar.
- Riesgos y mitigaciones: entrega at-least-once → consumidores idempotentes.

## Verificación
Cómo sabremos que la decisión se cumple (fitness function, test, métrica, alerta).
```

### Qué merece un ADR

- Estilo arquitectónico (monolito modular vs. microservicios).
- Modelo de datos multi-tenant (BD / esquema / fila + RLS).
- Estrategia de consistencia (Saga, Outbox, transacción local).
- Elección de un proveedor crítico (custodia de claves, pagos, identidad).
- Contratos públicos (versionado de API, formato de eventos).

**No** merecen ADR: decisiones fáciles de revertir (nombre de una variable, librería de
utilidades interna).

---

## 5. Fitness functions (arquitectura verificable)

Una decisión que no se verifica automáticamente se erosiona. Convierte las reglas en
pruebas que corren en CI:

| Regla arquitectónica | Fitness function |
|---|---|
| El dominio no depende de infraestructura | Test de dependencias (ArchUnit, dependency-cruiser, `eslint-plugin-boundaries`) |
| Toda query filtra por tenant | RLS activo + test de integración que consulta con otro tenant y espera 0 filas |
| Ningún pago se procesa dos veces | Test de concurrencia con la misma clave de idempotencia |
| Latencia p95 < 300 ms en `/checkout` | Test de carga en pipeline + alerta de SLO |
| Sin dependencias circulares entre módulos | `madge --circular` / análisis estático |
| Los contratos de eventos son compatibles hacia atrás | Schema registry o contract testing (Pact) |

---

## 6. Registro de riesgos técnicos

| Riesgo | Probabilidad | Impacto | Mitigación | Dueño | Disparador de revisión |
|---|---|---|---|---|---|
| Divergencia on-chain vs. fiat | Media | Alto | Saga + reconciliación periódica | Backend | > 0,1 % de transacciones en revisión manual |
| Fuga entre tenants | Baja | Crítico | RLS + ABAC en KMS + tests | Seguridad | Cualquier hallazgo de pentest |

---

## 7. Formato de entrega de una propuesta de arquitectura

1. **Resumen ejecutivo** (3–5 líneas): problema, decisión, costo principal.
2. **Drivers**: requisitos clave, restricciones y escenarios de calidad priorizados.
3. **Vistas C4**: contexto (nivel 1) y contenedores (nivel 2); componentes (nivel 3) solo
   donde haya complejidad real.
4. **Decisiones**: lista de ADR con estado.
5. **Trade-offs y riesgos**: qué se sacrifica y cómo se mitiga.
6. **Plan incremental**: qué se construye primero, qué se difiere y qué señal dispararía
   el siguiente paso (ej. "extraer el servicio X cuando supere N req/s").
