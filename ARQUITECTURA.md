# Arquitectura AndesStay (v2)

Alcance operativo: login Azure AD, JWT (Gateway + BFF), microservicios de dominio detrás del BFF, panel de auditoría alimentado por eventos Kafka, y mensajería RabbitMQ (publicador en reservations, consumidores en notify y microservicio administrador).

El navegador **solo** habla con Azure AD (login) y con el **API Gateway / BFF** (`/api`). Nunca llama directo a catálogo, reservas, audit ni notify.

```mermaid
flowchart LR
  U[Usuario]
  FE[Angular + MSAL]
  AAD[Azure AD / Entra ID]
  GW[API Gateway]
  BFF[BFF]
  CAT[Catálogo]
  RES[Reservas]
  AUD[Audit]
  KFK[Kafka]
  RMQ[RabbitMQ]
  NTF[Notify]
  ADM[Rabbit admin]

  U --> FE
  FE -->|login PKCE| AAD
  AAD -->|tokens| FE
  FE -->|HTTPS + Bearer JWT| GW
  GW -->|JWT + CORS| BFF
  BFF --> CAT
  BFF --> RES
  BFF --> AUD
  RES -->|reservations.events| KFK
  KFK -->|consumidor| AUD
  RES -.->|cmd.direct / cmd.topic| RMQ
  RMQ --> NTF
  ADM -.->|crea/elimina colas, exchanges, bindings| RMQ
```

## Flujo JWT (resumen)

1. Usuario abre Angular (`http://localhost:4200`).
2. MSAL → Azure AD (Authorization Code + PKCE, **sin** client secret).
3. Access token v2 con `roles` y scope `access_as_user`.
4. Interceptor / llamadas con `Authorization: Bearer` a `apiUrl` (Gateway o BFF local).
5. Gateway: CORS + JWT → sin token **401**.
6. BFF: issuer, audience, firma, exp + **roles** → sin permiso **403**.
7. BFF proxea a catálogo, reservas y audit según ruta.
8. Dominio **no** se expone al browser.

## Flujo mensajería (EP3)

1. Caso de uso asíncrono (ej. reserva confirmada) → productor publica a exchange `cmd.direct` / `cmd.topic`.
2. Colas: `q.cmd.email`, `q.cmd.housekeeping`, `q.cmd.voucher` (+ `*.dlq`).
3. `andesstay-ms-notify` consume con `@RabbitListener` (un consumidor por dominio) y ACK manual.
4. Decisión ante un fallo (`ReliableDelivery`):

   | Resultado | Acción |
   | --- | --- |
   | Éxito | ACK |
   | Mensaje duplicado (mismo `eventId`) | ACK sin ejecutar el trabajo |
   | Error recuperable (`RecoverableDeliveryException`) | Reintenta hasta `andesstay.notify.retry.max-attempts` con `backoff-ms`, log `RETRY attempt=x/N`; si se agotan → NACK sin requeue → DLQ |
   | Cualquier otro error | NACK sin requeue → DLQ en el primer fallo |

   Cada NACK queda en el log (`reasonKind=exhausted` o `non-recoverable`) y en `GET /api/notify/metrics/dlq`.
5. `andesstay-ms-rabbit-admin` (puerto `8085`): API REST para crear, actualizar y eliminar colas, exchanges y bindings (`/api/queues`, `/api/exchanges`, `/api/bindings`). Responde 201, 200, 204, 400 por validación y 404 al borrar un recurso inexistente. Toda la lógica vive en `RabbitAdminService`. **No tiene autenticación propia:** no debe exponerse por el Gateway, solo por la red interna. Hoy se ejecuta aparte; **todavía no figura en `compose.yml`** de infra.

Detalle de nombres: [notify README](https://github.com/HeOlivares/andesstay-ms-notify) · [rabbit-admin README](https://github.com/HeOlivares/andesstay-ms-rabbit-admin).

## Flujo de auditoría (Kafka)

1. `andesstay-ms-reservations` publica cada evento de reserva (creada o cambio de estado) en el topic `reservations.events`.
2. `andesstay-ms-audit` lo consume (`ReservationEventListener`) y guarda un registro de auditoría con acción, recurso, actor y detalle (`PENDING -> CONFIRMED`, unidad, huéspedes).
3. El panel `/auditoria` lee `GET /api/audit/events` y `GET /api/audit/metrics` a través del BFF (solo roles Auditor y Admin).
4. Las métricas se calculan sobre la tabla `reservations` de la base compartida: `ms-audit` necesita que `ms-reservations` ya haya creado el esquema.

## Repositorios

| Componente | Repo | Dueño |
| --- | --- | --- |
| Frontend | [andesstay-frontend](https://github.com/cristianmonsalve14/andesstay-frontend) | Cristian |
| Docs | [andesstay-docs](https://github.com/cristianmonsalve14/andesstay-docs) | Cristian |
| BFF | [andesstay-ms-bff](https://github.com/HeOlivares/andesstay-ms-bff) | Héctor |
| Catálogo | [andesstay-ms-catalog](https://github.com/HeOlivares/andesstay-ms-catalog) | Héctor |
| Notify | [andesstay-ms-notify](https://github.com/HeOlivares/andesstay-ms-notify) | Héctor |
| Rabbit admin | [andesstay-ms-rabbit-admin](https://github.com/HeOlivares/andesstay-ms-rabbit-admin) | Héctor |
| Reservas | [andesstay-ms-reservations](https://github.com/RolandoLillo/andesstay-ms-reservations) | Rolando |
| Audit | [andesstay-ms-audit](https://github.com/RolandoLillo/andesstay-ms-audit) | Rolando |
| Infra / Gateway | [andesstay-infra](https://github.com/RolandoLillo/andesstay-infra) | Rolando |

Kanban: [AndesStay — Kanban](https://github.com/users/cristianmonsalve14/projects/3).

## URLs locales (lab)

| Servicio | URL típica |
| --- | --- |
| Frontend | `http://localhost:4200` |
| BFF (lab Windows; 8080 a veces ocupado) | `http://localhost:8083` |
| Frontend `apiUrl` | `http://localhost:8083/api` (local) o Gateway en nube |
| Catálogo | detrás del BFF (`/api/catalog/...`) |
| Reservas | detrás del BFF (`/api/reservations/...`) |
| Audit | detrás del BFF (`/api/audit/...`) |
| Notify health | `GET /api/notify/health` (puerto `8083` en compose; en el lab local puede chocar con el BFF si ambos usan `8083`) |
| RabbitMQ AMQP / UI | `5672` / `15672` (clúster `rabbitmq-1` + `rabbitmq-2` en `compose.yml`) |
| Rabbit admin | `http://localhost:8085/api/queues` (se ejecuta aparte; no está en `compose.yml`) |

En EC2/compose del equipo el BFF suele publicarse en **8080**. No commitear URLs de lab personales en `main` del frontend.

## Roles → acceso (Caso 5 / producto)

| Rol | Catálogo | Reservas | Auditoría |
| --- | --- | --- | --- |
| Admin | ver + crear/editar | ver + crear + estado | sí |
| Operador | ver | ver + crear + estado | no |
| Cliente | 403 en BFF (no opera catálogo) | ver propias + crear | no |
| Auditor | no opera | no opera | sí |

## Matriz y evidencias

- [MATRIZ-FUNCIONALIDADES.md](MATRIZ-FUNCIONALIDADES.md)
- [evidencias/](evidencias/README.md)
- Identidad: [IDENTIDAD.md](IDENTIDAD.md)

## Fuera de este documento

- Client secret en App Registration (no aplica; PKCE).
- MS report / KPIs avanzados (parking lot semestre).
