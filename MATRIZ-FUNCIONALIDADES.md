# Matriz de funcionalidades — AndesStay

**Issue:** [andesstay-docs#3](https://github.com/cristianmonsalve14/andesstay-docs/issues/3)  
**Regla de puntaje EP3:** solo cuentan filas **implementada** con evidencia verificable (endpoint, commit/PR, prueba o captura). Parcial o sin evidencia = no cuenta.

Estados: `implementada` | `parcial` | `no implementada`

Última actualización: 2026-10-19

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
| F17 | Panel `/auditoria` (Admin / Auditor) | implementada | frontend + BFF + ms-audit | Ruta `auditoria`; proxy audit en BFF |
| F18 | Métricas / eventos de auditoría vía BFF | implementada | ms-audit + BFF | `GET /api/audit/metrics`, `GET /api/audit/events` |

## Infra y Gateway

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F19 | Docker Compose de apps (BFF, catalog, reservations, …) | implementada | andesstay-infra | [infra#9](https://github.com/RolandoLillo/andesstay-infra/pull/9) |
| F20 | API Gateway JWT + CORS (lab / EC2) | parcial | andesstay-infra / AWS | Receta en infra; URL de lab no fija en `main` |
| F21 | Frontend consume solo Gateway/BFF (no MS directo) | implementada | andesstay-frontend | `environment.apiUrl` → `/api` |

## EP3 — RabbitMQ / mensajería

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F22 | Topología Rabbit: exchanges, colas, bindings | implementada | andesstay-ms-notify | [notify README](https://github.com/HeOlivares/andesstay-ms-notify) — `cmd.direct`, `q.cmd.email`, etc. |
| F23 | Consumidores `@RabbitListener` por dominio + ACK/NACK | implementada | andesstay-ms-notify | Listeners email / housekeeping / voucher; ACK manual |
| F24 | DLQ + log de mensajes no entregados | implementada | andesstay-ms-notify | Colas `*.dlq`; `GET /api/notify/metrics/dlq` |
| F25 | Nombres de colas/exchanges centralizados en yml | parcial | andesstay-ms-notify | Hoy en `RabbitTopologyConfig` (constantes); falta yml (tarjeta Héctor H1) |
| F26 | Productores desde dominio (p. ej. confirmar reserva → cola) | no implementada | ms-reservations | Tarjeta Rolando R2 |
| F27 | RabbitMQ en docker-compose del equipo | no implementada | andesstay-infra | Tarjeta Rolando R1 |
| F28 | MS administrador REST (crear/eliminar colas, exchanges, bindings) | no implementada | ms-rabbit-admin (nuevo) | Tarjetas Héctor H2–H5 |
| F29 | `RabbitAdminService` + validación de entrada | no implementada | ms-rabbit-admin | Tarjetas Héctor H3–H5 |

## Reportes (fuera de EP3 inmediato)

| ID | Funcionalidad | Estado | Componente | Evidencia |
| --- | --- | --- | --- | --- |
| F30 | MS report / pantallas `/reports` | no implementada | futuro | Parking lot semestre |

---

## Conteo (para rúbrica de sistematización)

| Estado | Cantidad | Notas |
| --- | --- | --- |
| implementada | 22 | F01–F19, F21–F24 (F20 parcial no cuenta) |
| parcial | 2 | F20, F25 |
| no implementada | 6 | F26–F30 + contar F20/F25 fuera del numerador |

**Alcance considerado para EP3 (F01–F29):** 29 funcionalidades.  
**Implementadas operativas hoy:** 22 → **~76%** del alcance listado.  
**Para ≥90%:** cerrar F25–F29 (yml + compose Rabbit + publisher + admin REST/servicio/validación) y subir F20 si hay evidencia Gateway estable.

Cuando Héctor/Rolando mergeen, actualizar esta tabla y las capturas en `evidencias/`.
