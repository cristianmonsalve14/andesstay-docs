# Evidencias

Capturas y pruebas verificables para [docs#3](https://github.com/cristianmonsalve14/andesstay-docs/issues/3) y la [matriz](../MATRIZ-FUNCIONALIDADES.md).

**No subir** tokens, contraseñas ni `.pem`. En capturas, oculta el Bearer si aparece completo.

## Checklist JWT / roles (base EP1–EP2)

| Archivo sugerido | Qué debe verse | Estado |
| --- | --- | --- |
| `01-login.png` | Pantalla login / redirect Microsoft | capturada — subir PNG |
| `02-token-claims.png` | Access token o `/api/me` con `roles` | capturada — subir PNG |
| `03-api-me-200.png` | `GET /api/me` → 200 (Admin u otro rol) | capturada — subir PNG |
| `04-sin-token-401.png` | Sin Bearer → 401 | capturada — subir PNG |
| `05-cliente-catalog-403.png` | Cliente + `GET /api/catalog/units` → 403 | capturada — subir PNG |
| `06-logout.png` | Cierre de sesión Microsoft / post-logout | capturada — subir PNG |

Mientras no estén las PNG, la evidencia de código/PR de la matriz sigue valiendo; las capturas cierran el criterio visual del issue #3.

### Cómo sacarlas (local)

1. Frontend: `http://localhost:4200` · BFF: `http://localhost:8083` (en lab Windows; en compose/EC2 suele ser `8080`).
2. Incógnito sin login → `GET http://localhost:8083/api/me` → **401**.
3. Login Admin → Dashboard → Network ` /api/me` → **200** + roles.
4. Login Cliente → intentar catálogo → BFF **403** (o mensaje en UI).
5. Cerrar sesión → captura logout.

## Checklist EP3 (Rabbit) — capturas runtime (pendientes)

Código ya en `main` (matriz F25–F29). Falta solo PNG de demo:

| Archivo sugerido | Qué debe verse | Evidencia de código |
| --- | --- | --- |
| `10-rabbit-cluster.png` | `rabbitmqctl cluster_status` o UI Nodes: `rabbit@rabbitmq-1` + `rabbit@rabbitmq-2` | [infra#11](https://github.com/RolandoLillo/andesstay-infra/pull/11) |
| `11-rabbit-queues.png` | UI Management con colas `q.cmd.*` y `*.dlq` | [notify#12](https://github.com/HeOlivares/andesstay-ms-notify/pull/12) |
| `12-publish-consume.png` | Confirmar reserva → mensaje en colas / log notify | [reservations#13](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/13) |
| `13-dlq.png` | Mensaje en DLQ tras NACK / demo fail | [notify#15](https://github.com/HeOlivares/andesstay-ms-notify/pull/15) |
| `14-admin-create-queue.png` | `POST /api/queues` → 201 | [rabbit-admin#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7) |
| `15-admin-validation-400.png` | `POST /api/queues` con `"name":""` → 400 `{"error":"..."}` | [rabbit-admin#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7) |

### Cómo sacarlas (EC2 o local)

1. Infra: `cd apps && docker compose --env-file ../.env up -d --build --remove-orphans` (con `RABBITMQ_ERLANG_COOKIE` en `.env`).
2. Cluster: `docker exec andesstay-rabbitmq-1 rabbitmqctl cluster_status` → captura `10`.
3. UI `:15672` → Queues → captura `11`.
4. Confirmar una reserva (`CONFIRMADA` + `guestEmail`) → captura colas/log notify (`12`).
5. Demo DLQ notify si aplica → captura `13`.
6. rabbit-admin: crear cola OK (`14`); POST name vacío → 400 (`15`).

## Evidencia ya referenciada sin captura

- Issues frontend cerrados: #1–#7, #12–#20 (identidad, guards, dashboard, reservas, catálogo).
- Notify topología yml: [notify#12](https://github.com/HeOlivares/andesstay-ms-notify/pull/12); demo/DLQ: [notify#15](https://github.com/HeOlivares/andesstay-ms-notify/pull/15).
- Publisher reservas → Rabbit: [reservations#13](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/13).
- Infra compose + clúster Rabbit: [infra#9](https://github.com/RolandoLillo/andesstay-infra/pull/9), [infra#11](https://github.com/RolandoLillo/andesstay-infra/pull/11).
- rabbit-admin REST + validación: [rabbit-admin#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7).
