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

## Checklist EP3 (Rabbit) — cuando existan PRs

| Archivo sugerido | Qué debe verse | Depende de |
| --- | --- | --- |
| `10-rabbit-queues.png` | UI Management con colas `q.cmd.*` y `*.dlq` | Rolando R1 |
| `11-publish-consume.png` | Confirmar reserva → mensaje consumido / log notify | Rolando R2 |
| `12-dlq.png` | Mensaje en DLQ tras NACK | Héctor H6 / notify |
| `13-admin-create-queue.png` | `POST /api/queues` OK | Héctor H4 |
| `14-admin-delete-queue.png` | `DELETE /api/queues/{name}` OK | Héctor H4 |

## Evidencia ya referenciada sin captura

- Issues frontend cerrados: #1–#7, #12–#20 (identidad, guards, dashboard, reservas, catálogo).
- Notify: README + topología en [HeOlivares/andesstay-ms-notify](https://github.com/HeOlivares/andesstay-ms-notify).
- Infra compose: PR [#9](https://github.com/RolandoLillo/andesstay-infra/pull/9).
