# Patrones de Integración: Backend + Stellar Disbursement Platform + Privy + Anchors SEP

> **Alcance**: arquitectura de referencia para plataformas que ejecutan desembolsos
> on-chain sobre Stellar y liquidan en moneda local vía anchors SEP. El stack de ejemplo
> es **NestJS + Stellar Disbursement Platform (SDP) + Privy + anchor SEP-24**, pero los
> patrones aplican a cualquier backend.
>
> **Fuentes verificadas** (noviembre 2026): código fuente de
> [`stellar-disbursement-platform-backend`](https://github.com/stellar/stellar-disbursement-platform-backend)
> (rama `develop`), especificación [SEP-24](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0024.md),
> [documentación de la SDP](https://developers.stellar.org/docs/platforms/stellar-disbursement-platform)
> y [Privy — Tier 2 / raw sign](https://docs.privy.io/recipes/use-tier-2). La SDP evoluciona
> rápido: antes de fijar un contrato de integración, vuelve a confirmar endpoints, roles y
> eventos contra la versión que vas a desplegar.

Aplica este documento junto con `patrones_arquitectura_distribuida.md` (Saga, Outbox,
Circuit Breaker, multi-tenancy): ese documento es el marco general y este es su
especialización al dominio de desembolsos sobre Stellar.

---

## 1. Vista de contenedores (C4 nivel 2)

```
┌──────────────┐  HTTPS/JWT   ┌──────────────────┐  API Key +        ┌──────────────────────┐
│ Frontend web │ ───────────▶ │ Backend propio   │  SDP-Tenant-Name  │ SDP Backend          │
│ / móvil      │              │ (NestJS)         │ ────────────────▶ │ (API + scheduler)    │
└──────────────┘              │ - dominio        │ ◀──────────────── │ - outbox de eventos  │
                              │ - reconciliación │  webhook firmado  └─────────┬────────────┘
                              │ - outbox propio  │  (HMAC-SHA256)              │ cola en BD
                              └───────┬──────────┘                   ┌─────────▼────────────┐
                                      │ SEP-10/24 + polling          │ TSS (Transaction     │
                              ┌───────▼──────────┐                   │ Submission Service)  │
                              │ Anchor SEP-24    │ ◀── pago Stellar ─│ channel accounts     │
                              │ (off-ramp fiat)  │                   └─────────┬────────────┘
                              └──────────────────┘                             ▼
                                                                        Red Stellar
```

Fronteras de responsabilidad:

| Componente | Es dueño de | No es dueño de |
|---|---|---|
| Backend propio | Reglas de negocio, identidad del tenant, clave de idempotencia, conciliación contable | Firma y envío de pagos masivos (lo hace la SDP/TSS) |
| SDP | Ciclo de vida del desembolso y de cada pago, invitación a receptores, aprobación | Liquidación fiat |
| TSS | Sumisión a la red, reintentos, channel accounts | Reglas de negocio |
| Anchor SEP-24 | KYC (SEP-12), cotización (SEP-38), salida a rieles fiat locales | Estado del desembolso en la SDP |

---

## 2. Resolución de tenant frente a la SDP

La SDP aísla tenants en PostgreSQL con un esquema por tenant (`sdp_<tenant>`), un
esquema de administración y un esquema compartido para el TSS. La SDP resuelve el tenant
con esta **prioridad**: **JWT** (requests autenticados) → **header `SDP-Tenant-Name`** →
**subdominio** (ej. `tenant1.sdp.example.com`).

| Canal desde el backend | Mecanismo | Dónde se aplica |
|---|---|---|
| Usuario autenticado en el dashboard de la SDP | Claim de tenant en el JWT | Lo resuelve la SDP |
| Service-to-service (API key) | Header `SDP-Tenant-Name` | Interceptor del cliente HTTP hacia la SDP |
| Flujo interactivo SEP-24 / registro de receptores | Subdominio | Generación de deep links con el subdominio del tenant |

**Invariante de diseño:** el tenant de la SDP se deriva 1:1 del identificador de tenant
del dominio propio (ej. `organization_id`) **en un único punto** y se propaga desde ahí.
Si dos canales pueden producir tenants distintos para la misma operación, existe una
vulnerabilidad de **cross-tenant contamination** (ver sección 6).

---

## 3. Modelo de estados

### 3.1 Desembolso (lote)

`DRAFT` → `READY` → `STARTED` → `COMPLETED`, con `PAUSED` (reanudable) y `CANCELED`.
Pasar a `STARTED` es la **aprobación**: se hace con `PATCH /disbursements/{id}/status`.

### 3.2 Pago individual (máquina de estados del código fuente)

| Estado | Terminal | Significado |
|---|---|---|
| `DRAFT` | No | Registrado; el desembolso no ha iniciado |
| `READY` | No | Esperando que el receptor registre su wallet |
| `PENDING` | No | Marcado para sumisión a Stellar (puede o no haberse enviado) |
| `PAUSED` | No | Pausado junto con su desembolso; reanudable |
| `SUCCESS` | Sí | Confirmado en la red |
| `FAILED` | No* | Falló al enviarse; **se puede reintentar** (`PATCH /payments/retry` → `PENDING`) |
| `CANCELED` | Sí | Cancelado manual o automáticamente (`ready_payments_cancellation_job`) |

Transiciones válidas: `DRAFT→READY`, `READY→PENDING|PAUSED|CANCELED|FAILED`,
`PAUSED→READY`, `PENDING→SUCCESS|FAILED`, `FAILED→PENDING`.

### 3.3 Transacción SEP-24 (especificación oficial)

`incomplete`, `pending_user_transfer_start`, `pending_user_transfer_complete` (solo
retiros), `pending_external`, `pending_anchor`, `on_hold`, `pending_stellar`,
`pending_trust`, `pending_user`, y los terminales **`completed`**, **`refunded`**,
**`expired`**, `no_market`, `too_small`, `too_large`. Además, `error` indica fallo.

> No existen `success`, `failed` ni `returned` en SEP-24. Mapear con esos nombres es un
> error común que rompe la reconciliación.

---

## 4. Patrones de comunicación: síncrono vs. asíncrono

| Operación | Tipo | Endpoint / canal SDP | Roles autorizados |
|---|---|---|---|
| Crear desembolso | Síncrono (ingestión) | `POST /disbursements` + `POST /disbursements/{id}/instructions` (CSV) | owner, financial_controller, initiator |
| Aprobar / iniciar | Síncrono (cambio de estado) | `PATCH /disbursements/{id}/status` | owner, financial_controller, approver |
| Pago directo | Síncrono | `POST /payments` | owner, financial_controller |
| Reintentar pagos fallidos | Síncrono | `PATCH /payments/retry` | owner, financial_controller |
| Progreso | Asíncrono (polling) | `GET /statistics`, `GET /statistics/{id}`, `GET /payments?…` | cualquier rol |
| Cambios de estado | Asíncrono (push) | Webhook de eventos de la SDP | — |

**Implicación clave**: una respuesta `201`/`200` confirma **ingestión o cambio de estado
solicitado**, no **ejecución en la red**. El backend no marca un movimiento como
confirmado hasta que el pago llega a `SUCCESS`.

### 4.1 Webhooks de la SDP (outbox + HMAC)

La SDP escribe eventos en una tabla outbox **dentro de la misma transacción** que el
cambio de estado, y el `event_delivery_job` los entrega cada ~30 s por HTTP POST
(lotes de 100, hasta 20 intentos; los envíos que fallan se quedan en la tabla para
operación). Eventos: `disbursement.created|approved|rejected|submitted|completed|failed|paused`
y `wallet.created|archived|promoted_to_default|membership_granted|membership_revoked`.

Cada envío lleva `X-SDP-Timestamp` y `X-SDP-Signature: sha256=<HMAC-SHA256>` con el
secreto de webhook de la organización. El receptor debe:

1. Verificar la firma con comparación en tiempo constante y rechazar timestamps viejos
   (protección contra replay).
2. Ser **Idempotent Consumer**: la entrega es *at-least-once*, así que pueden llegar
   duplicados.
3. Responder `2xx` rápido y procesar en segundo plano (encolar).

### 4.2 Jobs internos de la SDP (no se reimplementan: se consume su efecto)

- `send_receiver_wallets_sms_invitation_job`: invita a receptores sin wallet registrada.
- `stellar_payment_to_submitter_job` / `circle_payment_to_submitter_job`: empuja pagos
  aprobados al TSS (o a Circle, si la cuenta de distribución es Circle).
- `payment_from_submitter_job`: lee la resolución on-chain y actualiza `SUCCESS`/`FAILED`.
- `circle_reconciliation_job`: concilia pagos enviados por Circle.
- `ready_payments_cancellation_job`: cancela pagos `READY` que vencen sin registro.
- `event_delivery_job`: entrega los webhooks (sección 4.1).

---

## 5. Reconciliación dual (on-chain + off-ramp fiat)

Cuando un desembolso cruza **dos sistemas de estado independientes** —el pago SDP en
Stellar y la transacción SEP-24 del anchor— pueden divergir: el lado on-chain tiene
éxito mientras el fiat falla, o viceversa.

### Tabla de reconciliación

| Pago SDP | Transacción SEP-24 | Estado reconciliado | Acción |
|---|---|---|---|
| `PENDING` | `pending_user_transfer_start` | En tránsito en la red | Esperar; el TSS reintenta la sumisión |
| `FAILED` | `pending_user_transfer_start` | Fallo on-chain | El anchor no recibió fondos: reintento seguro (`PATCH /payments/retry`) |
| `SUCCESS` | `pending_anchor` / `pending_external` / `on_hold` | Procesando fiat | **No reintentar on-chain**; solo monitorear |
| `SUCCESS` | `pending_user_transfer_complete` | Fondos fiat listos para retiro | Notificar al beneficiario |
| `SUCCESS` | `completed` | Completado | Cerrar con la clave de idempotencia definitiva |
| `SUCCESS` | `refunded` | **Reversa** | El anchor devolvió los fondos on-chain → asiento de compensación append-only (nunca editar el original) |
| `SUCCESS` | `error` / `expired` | **Excepción operativa** | Abrir caso manual; consultar `refunds` en la transacción del anchor antes de actuar |

> Es el problema que resuelve una **Saga orquestada** (ver
> `patrones_arquitectura_distribuida.md`): cada paso (envío on-chain, salida fiat) es una
> transacción local con su compensación definida.

### Idempotencia: dos mecanismos complementarios

1. **Clave de idempotencia de negocio**: cada solicitud de desembolso lleva un UUID
   generado por el backend *antes* de tocar la SDP o el anchor; un reintento de red no
   produce un doble desembolso. Persístela con una restricción `UNIQUE`.
2. **Memo de Stellar**: para asociar un pago on-chain con la orden del anchor se usa el
   memo que el anchor entrega (`withdraw_memo` + `withdraw_memo_type`, típicamente `id`
   de 64 bits o `hash`). Usa exactamente el tipo que indica el anchor.

### Esqueleto de servicio de reconciliación (NestJS)

```typescript
const FINAL_OK = new Set(['completed']);
const FINAL_REFUND = new Set(['refunded']);
const NEEDS_REVIEW = new Set(['error', 'expired', 'no_market', 'too_small', 'too_large']);

@Injectable()
export class ReconciliationService {
  private readonly logger = new Logger(ReconciliationService.name);

  // Respaldo: el canal primario es el webhook/callback; esto atrapa lo que se pierda.
  @Cron(CronExpression.EVERY_5_MINUTES)
  async reconcilePendingOffRamps() {
    const pending = await this.transfers.findAwaitingAnchor();
    for (const transfer of pending) {
      try {
        const tx = await this.anchor.getTransaction(transfer.tenantId, transfer.anchorTxId);
        await this.apply(transfer, tx.status);
      } catch (err) {
        this.logger.error(`Reconciliación fallida para ${transfer.id}`, err);
      }
    }
  }

  private async apply(transfer: Transfer, status: string) {
    if (FINAL_OK.has(status)) return this.transfers.markCompleted(transfer.id);
    if (FINAL_REFUND.has(status)) return this.ledger.recordCompensation(transfer.id);
    if (NEEDS_REVIEW.has(status)) return this.cases.openManualReview(transfer.id, status);
    // estados pending_*: no hacer nada, seguir esperando
  }
}
```

`anchor` es un puerto (`OffRampProvider`) con un adaptador por anchor: el servicio no
conoce URLs, tokens SEP-10 ni formatos de cada proveedor.

---

## 6. Custodia de firmas en multi-tenant: riesgos y mitigaciones

La SDP firma con cuentas de distribución (Stellar o Circle) cuyas claves se cifran con
una passphrase. Mover el material criptográfico a un KMS/HSM mitiga la exposición, pero
**no elimina los riesgos lógicos de diseño** en multi-tenant.

### Riesgos

1. **Cross-tenant contamination**: con un único rol IAM global frente al KMS, un fallo en
   la resolución de tenant permite que un tenant obtenga firmas con la clave de otro.
2. **Baja de tenant sin proceso**: deja claves huérfanas o desvincula recursos de forma
   irreversible.
3. **Ausencia de límites redundantes**: si todo el control es "quién puede invocar
   `kms:Sign`", unas credenciales cloud comprometidas producen firmas válidas hasta que
   alguien lo detecte.

> El KMS resuelve que las claves no salgan del perímetro de custodia; **no** resuelve el
> aislamiento entre tenants dentro del backend. Son dos problemas con controles distintos.

### Mitigaciones

- **Una clave por tenant** en el KMS, con key policy propia; nunca una clave compartida.
- **ABAC**: credenciales temporales de STS con tag `TenantID` y key policy que lo exige
  antes de autorizar `kms:Sign`. Defensa en profundidad, igual que RLS en PostgreSQL.
- **Límites on-chain**: cuenta de distribución con saldo acotado (fondeo por tramos) y,
  donde aplique, multisig con umbrales para operaciones de tesorería.
- **Segregación de funciones** (ver matriz).

### Matriz de roles (código fuente de la SDP)

| Rol | Alcance | Puede | Nota |
|---|---|---|---|
| `owner` | Todo el tenant | Administración, usuarios, crear y aprobar | No usarlo para la operación financiera diaria |
| `financial_controller` | Por wallet | Crear, aprobar, pagos directos, reintentos | Rol financiero completo |
| `initiator` | Por wallet | Crear desembolsos e instrucciones | **No** puede aprobar |
| `approver` | Por wallet | Cambiar el estado (aprobar/iniciar, pausar, cancelar) | **No** puede crear |
| `business` | Por wallet | Lectura de operación | Sin escritura financiera |
| `developer` | Todo el tenant | API keys, configuración técnica | Fuera de los flujos financieros |

**Principio de cuatro ojos:** activa `is_approval_required` en la organización. Con ese
flag, la SDP rechaza (`ErrDisbursementStartedByCreator`) que el mismo usuario que creó el
desembolso lo inicie. Sin el flag, un `owner` o `financial_controller` puede crear y
aprobar solo: no confíes en los roles por sí solos.

---

## 7. Firma con Privy

Privy soporta Stellar como cadena **Tier 2**: genera y custodia claves Ed25519 y expone
**raw sign** (firma un hash o bytes que le entregas). No construye ni envía transacciones
Stellar. El backend:

1. Construye la transacción con el Stellar SDK.
2. Calcula el hash de firma (`transaction.hash()`, que incluye el network passphrase).
3. Pide a Privy el raw sign de ese hash.
4. Añade la firma como `DecoratedSignature` con el hint de la clave pública y envía la
   transacción a la red.

Encapsula esto detrás de un puerto `Signer` para poder cambiar Privy por KMS, un HSM u
otro proveedor sin tocar la lógica de negocio.

| Componente | Tipo | Custodia | Responsabilidad |
|---|---|---|---|
| **Stellar Wallet SDK** (SDF) | Cliente | Depende de la app | SEP-10 (auth) y flujo interactivo SEP-24 |
| **Privy** | Firmador (raw sign) | Gestionada por Privy (TEE + key sharding) | Firma; no construye XDR |
| **SDP** | Plataforma de desembolsos | Cuentas de distribución del tenant | Ciclo de vida de pagos masivos |

---

## 8. Selección de anchor off-ramp

Criterios para elegir un anchor de salida a fiat:

- **Rieles locales**: SPEI (México), Pix (Brasil), transferencias bancarias locales, SWIFT.
- **SEP-38** (cotizaciones firmes) para fijar el tipo USDC → moneda local antes de ejecutar.
- **SEP-12** (KYC) delegado al anchor, para no reimplementar cumplimiento.
- **Callbacks** de cambio de estado, para no depender solo de polling.
- **Tarifas**, límites (`min_amount`/`max_amount`) y tiempos de liquidación.
- **Vigencia**: que el anchor siga activo en el directorio de anchors de Stellar al
  integrar.

> Integra cada anchor detrás de un puerto `OffRampProvider` (patrón **Adapter**): cambiar
> de proveedor cambia una implementación, no el flujo de reconciliación. Ver *Vendor
> Lock-In por diseño* en `anti_patrones.md`.

---

## 9. Checklist de revisión arquitectónica

- [ ] El tenant se resuelve en un único punto y se propaga igual por todos los canales
      hacia la SDP (JWT, `SDP-Tenant-Name`, subdominio).
- [ ] Ningún movimiento se marca confirmado por la respuesta síncrona; se espera `SUCCESS`.
- [ ] La clave de idempotencia (UUID) se genera antes de tocar la SDP o el anchor y tiene
      restricción `UNIQUE`.
- [ ] Los estados SEP-24 se mapean con los nombres de la especificación (`completed`,
      `refunded`, `error`, `expired`…), no con nombres inventados.
- [ ] `refunded` genera un asiento de compensación append-only; `error`/`expired` abren
      revisión manual.
- [ ] El receptor de webhooks valida HMAC + timestamp y es idempotente.
- [ ] `is_approval_required` está activo, y initiator ≠ approver para montos relevantes.
- [ ] Cada tenant tiene su propia clave en el KMS, con ABAC sobre `TenantID`.
- [ ] Firmador (`Signer`) y anchor (`OffRampProvider`) están detrás de puertos propios.
- [ ] La cuenta de distribución tiene saldo acotado y alertas de consumo.
