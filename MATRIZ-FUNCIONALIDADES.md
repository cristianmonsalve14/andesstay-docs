# Matriz de funcionalidades — AndesStay

**Issue:** [andesstay-docs#3](https://github.com/cristianmonsalve14/andesstay-docs/issues/3)  
**Regla de puntaje EP3:** solo cuentan filas **implementada** con evidencia verificable (endpoint, commit/PR, prueba o captura). Parcial o sin evidencia = no cuenta.

Estados: `implementada` | `parcial` | `no implementada`

Última actualización: 2026-10-27

---

## Identidad y frontend

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F01 | Tenant Azure AD + usuarios de prueba por rol | implementada | Azure AD / docs | [frontend#1](https://github.com/cristianmonsalve14/andesstay-frontend/issues/1), [IDENTIDAD.md](IDENTIDAD.md) |
| F02 | App Registration SPA + scope `access_as_user` + PKCE (sin secret) | implementada | Azure AD / frontend | [frontend#2](https://github.com/cristianmonsalve14/andesstay-frontend/issues/2), [IDENTIDAD.md](IDENTIDAD.md) |
| F03 | App roles Admin / Operador / Cliente / Auditor | implementada | Azure AD | [frontend#3](https://github.com/cristianmonsalve14/andesstay-frontend/issues/3) |
| F04 | Angular + MSAL Authorization Code + PKCE | implementada | andesstay-frontend | [frontend#4](https://github.com/cristianmonsalve14/andesstay-frontend/issues/4), [frontend#12](https://github.com/cristianmonsalve14/andesstay-frontend/issues/12) |
| F05 | Guards por rol + `MsalInterceptor` / Bearer | implementada | andesstay-frontend | [frontend#5](https://github.com/cristianmonsalve14/andesstay-frontend/issues/5), [frontend#13](https://github.com/cristianmonsalve14/andesstay-frontend/issues/13) |
| F06 | Login / logout | implementada | andesstay-frontend | Rutas `/login`; ver [evidencias/README.md](evidencias/README.md) |
| F07 | Dashboard consume `GET /api/me` | implementada | andesstay-frontend | [frontend#14](https://github.com/cristianmonsalve14/andesstay-frontend/issues/14) |
| F08 | Vistas por rol (Cliente / Operador / Auditor / Admin) | implementada | andesstay-frontend | [frontend#17](https://github.com/cristianmonsalve14/andesstay-frontend/issues/17)–[#20](https://github.com/cristianmonsalve14/andesstay-frontend/issues/20) |

## BFF, catálogo y reservas

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F09 | BFF valida JWT (issuer, audience, firma, exp) | implementada | andesstay-ms-bff | Repo Héctor; endpoint `GET /api/me` → 401 sin token |
| F10 | Autorización por rol (401 / 403) | implementada | andesstay-ms-bff | Cliente → `GET /api/catalog/**` = 403; ver evidencias |
| F11 | Proxy BFF → catálogo / reservas | implementada | andesstay-ms-bff | Consumo desde Angular vía `/api/...` |
| F12 | Catálogo: listar unidades (tarifa, disponibilidad) | implementada | catalog + frontend | [frontend#16](https://github.com/cristianmonsalve14/andesstay-frontend/issues/16), MS catalog |
| F13 | Catálogo: Admin crea/edita unidades | implementada | catalog + frontend | [frontend#16](https://github.com/cristianmonsalve14/andesstay-frontend/issues/16); Operador solo consulta |
| F14 | Reservas: listar / crear / cambiar estado por rol | implementada | reservations + frontend | [frontend#15](https://github.com/cristianmonsalve14/andesstay-frontend/issues/15) |
| F15 | Reservas enrich (`unitName`, `guests`, `customerName`) | implementada | ms-reservations + frontend | [reservations#11](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/11) |
| F16 | Disponibilidad de unidad al confirmar reserva | implementada | ms-catalog | [catalog#10](https://github.com/HeOlivares/andesstay-ms-catalog/pull/10) |

## Auditoría

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F17 | Panel `/auditoria` (Admin / Auditor) | implementada | frontend + BFF + ms-audit | Ruta `auditoria` con roles `Auditor`/`Admin` en [`app.routes.ts`](https://github.com/cristianmonsalve14/andesstay-frontend/blob/main/src/app/app.routes.ts); proxy `GET /api/audit/**` en [`ProxyController`](https://github.com/HeOlivares/andesstay-ms-bff/blob/main/src/main/java/cl/duoc/andesstay/bff/web/ProxyController.java) (el BFF solo lo permite a Auditor/Admin) |
| F18 | Métricas / eventos de auditoría vía BFF | implementada | ms-audit + BFF | `GET /api/audit/metrics`, `GET /api/audit/events`; pruebas [`AuditControllerTest`](https://github.com/RolandoLillo/andesstay-ms-audit/blob/master/src/test/java/com/andesstay/audit/controller/AuditControllerTest.java) ([audit#3](https://github.com/RolandoLillo/andesstay-ms-audit/pull/3)); consumidor Kafka `ReservationEventListener` con prueba unitaria del JSON real del productor |

## Infra y Gateway

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F19 | Docker Compose de apps (BFF, catalog, reservations, …) | implementada | andesstay-infra | [infra#9](https://github.com/RolandoLillo/andesstay-infra/pull/9) |
| F20 | API Gateway JWT + CORS (lab / EC2) | parcial | andesstay-infra / AWS | Plantilla en `main` con rutas explícitas, JWT, audience sin `api://` y reenvío de la ruta completa ([infra#14](https://github.com/RolandoLillo/andesstay-infra/pull/14)). **Parcial hasta tener la prueba real en AWS** (health 200, `/api/me` 401/200, catálogo con Cliente 403, ruta inexistente 404) |
| F21 | Frontend consume solo Gateway/BFF (no MS directo) | implementada | andesstay-frontend | Todas las llamadas HTTP salen de [`bff-api.service.ts`](https://github.com/cristianmonsalve14/andesstay-frontend/blob/main/src/app/api/bff-api.service.ts) con `${environment.apiUrl}/...` (`apiUrl` → `/api`, ver [`environment.ts`](https://github.com/cristianmonsalve14/andesstay-frontend/blob/main/src/environments/environment.ts)); el navegador no llama a catálogo, reservas ni audit directamente |

## EP3 — RabbitMQ / mensajería

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F22 | Topología Rabbit: exchanges, colas, bindings (beans explícitos por caso de uso) | implementada | andesstay-ms-notify | [notify README](https://github.com/HeOlivares/andesstay-ms-notify) — `cmd.direct`, `q.cmd.email`, etc.; beans por flujo en [notify#20](https://github.com/HeOlivares/andesstay-ms-notify/pull/20) (`ExchangeTopologyConfig`, `Email/Housekeeping/VoucherTopologyConfig`) |
| F23 | Consumidores `@RabbitListener` por dominio + ACK/NACK | implementada | andesstay-ms-notify | Listeners email / housekeeping / voucher; ACK manual; reintentos ante errores recuperables y NACK diferenciado en [notify#18](https://github.com/HeOlivares/andesstay-ms-notify/pull/18) (tests `ReliableDeliveryTest`) |
| F24 | DLQ + log de mensajes no entregados | implementada | andesstay-ms-notify | Colas `*.dlq`; `GET /api/notify/metrics/dlq`; logs `RETRY attempt=x/N` y `NACK without requeue ... reasonKind=exhausted\|non-recoverable` ([notify#18](https://github.com/HeOlivares/andesstay-ms-notify/pull/18)) |
| F25 | Nombres de colas/exchanges centralizados en yml | implementada | andesstay-ms-notify + ms-reservations | [notify#12](https://github.com/HeOlivares/andesstay-ms-notify/pull/12) — topología en `application.yml`; publicador de reservations con `@ConfigurationProperties` y test de enlace ([reservations#15](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/15)) |
| F26 | Productores desde dominio (p. ej. confirmar reserva → cola) | implementada | ms-reservations | [reservations#13](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/13) — `NotificationEnvelope` al confirmar (`CONFIRMADA`), best-effort |
| F27 | RabbitMQ en docker-compose del equipo | implementada | andesstay-infra | [infra#11](https://github.com/RolandoLillo/andesstay-infra/pull/11) — clúster `rabbitmq-1` + `rabbitmq-2` |
| F28 | MS administrador REST (crear/eliminar colas, exchanges, bindings) | implementada | ms-rabbit-admin | [rabbit-admin#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7) — `/api/queues`, `/api/exchanges`, `/api/bindings`; 404 al borrar un recurso inexistente en [rabbit-admin#9](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/9) |
| F29 | `RabbitAdminService` + validación de entrada | implementada | ms-rabbit-admin | [rabbit-admin#3](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/3) + [#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7) — servicio + DTO `@Valid` → 400; 404 en [#9](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/9) |

## Reportes (fuera de EP3 inmediato)

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F30 | MS report / pantallas `/reports` | no implementada | futuro | Parking lot semestre |

---

## Conteo (para rúbrica de sistematización)

| Estado | Cantidad | Notas |
| --- | --- | --- |
| implementada | 28 | F01–F19, F21–F29 (F20 parcial no cuenta) |
| parcial | 1 | F20 |
| no implementada | 1 | F30 (reportes; fuera de EP3 inmediato) |

Fórmula de la rúbrica: (funcionalidades implementadas y operativas ÷ total del alcance) × 100.

| Total considerado | Cálculo | Resultado |
| --- | --- | --- |
| Alcance EP3 (F01–F29) | 28 ÷ 29 | **96,6 %** |
| Todas las filas de la matriz (F01–F30) | 28 ÷ 30 | **93,3 %** |

En ambos casos supera el 90 % que pide la rúbrica. Tomando el total más exigente (30), el mínimo es 27 filas implementadas: el margen es de **1 fila**. Por eso cada fila `implementada` debe mantener evidencia verificable.

**Pendiente:** F20 pasa a `implementada` solo cuando exista la prueba real del Gateway en AWS (ver su fila). Capturas de ejecución de EP3 en `evidencias/` (clúster, publicar/consumir, DLQ, administrador 201/400/404, panel de auditoría).
