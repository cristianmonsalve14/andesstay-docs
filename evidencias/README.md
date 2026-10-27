# Evidencias

Capturas y pruebas verificables para [docs#3](https://github.com/cristianmonsalve14/andesstay-docs/issues/3) y la [matriz](../MATRIZ-FUNCIONALIDADES.md).

**No subir** tokens, contraseñas ni `.pem`. En capturas, oculta el Bearer si aparece completo.

## Checklist JWT / roles (base EP1–EP2)

| Archivo | Qué debe verse | Estado |
| --- | --- | --- |
| `01-login.png` | Pantalla login / redirect Microsoft | subida |
| `02-token-claims.png` | Dashboard con rol y petición `me` → 200 en Network | subida |
| `03-api-me-200.png` | `GET /api/me` → 200 (Admin) | subida |
| `04-sin-token-401.png` | Sin Bearer → 401 | subida |
| `05-cliente-catalog-403.png` | Cliente + `GET /api/catalog/units` → 403 | **reemplazar**: la captura actual imprime el Bearer completo en la consola (repo público) |
| `06-logout.png` | Cierre de sesión Microsoft / post-logout | subida |

**Regla para las capturas:** nunca imprimir el token en la consola. Para mostrar el 403 usar **DevTools → Network** (petición `units` → Headers/Response) y, si el `Authorization` aparece, taparlo antes de guardar la imagen.

La evidencia de código/PR de la matriz sigue siendo la base; las capturas cierran el criterio visual del issue #3.

### Cómo sacarlas (local)

1. Frontend: `http://localhost:4200` · BFF: `http://localhost:8083` (en lab Windows; en compose/EC2 suele ser `8080`).
2. Incógnito sin login → `GET http://localhost:8083/api/me` → **401**.
3. Login Admin → Dashboard → Network `/api/me` → **200** + roles.
4. Login Cliente → intentar catálogo → BFF **403** (mirarlo en Network, no en la consola con el token).
5. Cerrar sesión → captura logout.

## Checklist EP3 (Rabbit, Kafka y Gateway) — capturas runtime (pendientes)

El código de EP3 está en `main`. Falta el PNG de cada demo:

| Archivo sugerido | Qué debe verse | Evidencia de código |
| --- | --- | --- |
| `10-rabbit-cluster.png` | `rabbitmqctl cluster_status` o UI Nodes: `rabbit@rabbitmq-1` + `rabbit@rabbitmq-2` | [infra#11](https://github.com/RolandoLillo/andesstay-infra/pull/11) |
| `11-rabbit-queues.png` | UI Management con colas `q.cmd.*` y `*.dlq` | [notify#12](https://github.com/HeOlivares/andesstay-ms-notify/pull/12), [notify#20](https://github.com/HeOlivares/andesstay-ms-notify/pull/20) |
| `12-publish-consume.png` | Confirmar reserva → mensaje en colas / log notify | [reservations#13](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/13) |
| `13-dlq.png` | Mensaje en DLQ tras NACK (`POST /api/notify/demo/fail-email`) | [notify#15](https://github.com/HeOlivares/andesstay-ms-notify/pull/15) |
| `14-admin-create-queue.png` | `POST /api/queues` → 201 | [rabbit-admin#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7) |
| `15-admin-validation-400.png` | `POST /api/queues` con `"name":""` → 400 `{"error":"..."}` | [rabbit-admin#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7) |
| `16-admin-delete-404.png` | `DELETE /api/queues/no-existe` → 404 `{"error":"Queue not found: ..."}` | [rabbit-admin#9](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/9) |
| `17-retry-ack.png` | Salida de `mvn test -Dtest=ReliableDeliveryTest` con `RETRY attempt=1/3 … 3/3`, `NACK … reasonKind=exhausted` y los 11 tests en verde | [notify#18](https://github.com/HeOlivares/andesstay-ms-notify/pull/18) |
| `18-panel-auditoria.png` | Panel `/auditoria` con un usuario Auditor o Admin | [audit#3](https://github.com/RolandoLillo/andesstay-ms-audit/pull/3) |
| `19-kafka-auditoria.png` | Crear/cambiar una reserva y ver el evento nuevo en `GET /api/audit/events` (prueba de punta a punta) | [audit#3](https://github.com/RolandoLillo/andesstay-ms-audit/pull/3) |
| `20-gateway-curls.png` | Resultados reales contra el Gateway: health 200, `/api/me` 401 y 200, catálogo con Cliente 403, ruta inexistente 404 | [infra#14](https://github.com/RolandoLillo/andesstay-infra/pull/14) |

Notas:
- `13-dlq` demuestra el camino de **error no recuperable** (mensaje inválido → DLQ directo). Los **reintentos** ante un error recuperable solo se ejercitan en los tests (`17`): hoy ningún sender real lanza `RecoverableDeliveryException`.
- `16` y `14`/`15` requieren `ms-rabbit-admin` en ejecución. **Aún no está en `compose.yml`**: se levanta aparte (puerto `8085`).
- `19` y `20` dependen de la prueba que debe ejecutar Rolando (Kafka local y Gateway en AWS). No marcar F20 como `implementada` sin `20`.
- Ocultar tokens y contraseñas de RabbitMQ en todas las imágenes.

### Cómo sacarlas (EC2 o local)

1. Infra: `cd apps && docker compose --env-file ../.env up -d --build --remove-orphans` (con `RABBITMQ_ERLANG_COOKIE` en `.env`).
2. Cluster: `docker exec andesstay-rabbitmq-1 rabbitmqctl cluster_status` → captura `10`.
3. UI `:15672` → Queues → captura `11`.
4. Confirmar una reserva (`CONFIRMADA` + `guestEmail`) → captura colas/log notify (`12`).
5. `POST /api/notify/demo/fail-email` → mensaje en `q.cmd.email.dlq` → captura `13`.
6. rabbit-admin (`.\mvnw.cmd spring-boot:run`, puerto `8085`): crear cola OK (`14`); POST name vacío → 400 (`15`); DELETE de una cola inexistente → 404 (`16`).
7. notify: `mvn test -Dtest=ReliableDeliveryTest` → captura de la consola (`17`).
8. Auditoría: panel `/auditoria` (`18`); crear y cambiar de estado una reserva y comprobar el evento en `/api/audit/events` (`19`).
9. Gateway: los curls de la fila `20` (`API=https://<API_ID>.execute-api.<region>.amazonaws.com/prod`; `curl -s -o NUL -w "%{http_code}\n" ...`), con el token en una variable de entorno y sin imprimirlo (`20`).

## Evidencia ya referenciada sin captura

- Issues frontend cerrados: #1–#7, #12–#20 (identidad, guards, dashboard, reservas, catálogo).
- Notify topología yml: [notify#12](https://github.com/HeOlivares/andesstay-ms-notify/pull/12); beans por caso de uso: [notify#20](https://github.com/HeOlivares/andesstay-ms-notify/pull/20); ACK y reintentos: [notify#18](https://github.com/HeOlivares/andesstay-ms-notify/pull/18); demo/DLQ: [notify#15](https://github.com/HeOlivares/andesstay-ms-notify/pull/15).
- Publisher reservas → Rabbit: [reservations#13](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/13); nombres centralizados: [reservations#15](https://github.com/RolandoLillo/andesstay-ms-reservations/pull/15).
- Infra compose + clúster Rabbit: [infra#9](https://github.com/RolandoLillo/andesstay-infra/pull/9), [infra#11](https://github.com/RolandoLillo/andesstay-infra/pull/11); Gateway: [infra#14](https://github.com/RolandoLillo/andesstay-infra/pull/14).
- rabbit-admin REST + validación: [rabbit-admin#7](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/7); 404: [rabbit-admin#9](https://github.com/HeOlivares/andesstay-ms-rabbit-admin/pull/9).
- Auditoría (consumidor Kafka + tests): [audit#3](https://github.com/RolandoLillo/andesstay-ms-audit/pull/3).
