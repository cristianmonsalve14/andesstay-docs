# Guion de presentación EP3 — AndesStay

> Borrador para revisar entre las tres personas del equipo. Los repartos de voz y las fechas están por definir. No incluye ningún secreto: los tokens y contraseñas se usan solo por variables de entorno y no se muestran en pantalla.

**Idea central (30 s):** AndesStay incorpora RabbitMQ para tareas asíncronas (notificación, ticket de housekeeping y voucher) sin tocar la lógica de reservas. Si algo falla, el mensaje no se pierde: va a una cola de mensajes no entregados (DLQ) y queda registrado en el log.

## 1. Recorrido por la rúbrica

| Indicador | Qué decir | Dónde mostrarlo |
| --- | --- | --- |
| Nombres centralizados | Los nombres de exchanges, colas y routing keys viven en un solo lugar por servicio. | `notify`: `application.yml`, bloque `andesstay.notify.topology.*`. `reservations`: `application.properties`, claves `rabbitmq.*`, leídas por `RabbitMessagingProperties` |
| Beans de Queue, Exchange y Binding | Cada caso de uso tiene sus beans, con su cola, su DLQ y sus bindings. | `notify`, paquete `config`: `ExchangeTopologyConfig`, `EmailTopologyConfig`, `HousekeepingTopologyConfig`, `VoucherTopologyConfig` |
| Configuración separada del negocio | Los servicios de negocio solo usan componentes ya configurados. | Mismas clases `*TopologyConfig`, `NotifyRetryProperties` y `AmqpMessageConverterConfig` |
| `@RabbitListener` por dominio | Un consumidor por dominio: email, housekeeping y voucher. | `notify`, paquete `messaging` |
| ACK y manejo de errores | ACK manual. Duplicado: ACK sin trabajar. Error recuperable: reintenta N veces con backoff y luego DLQ. Otro error: DLQ directo. | `ReliableDelivery`; tabla de decisiones en el README de `notify`; tests `ReliableDeliveryTest` |
| Endpoints REST del administrador | Crear, actualizar y eliminar colas, exchanges y bindings. | `rabbit-admin`: `/api/queues`, `/api/exchanges`, `/api/bindings` |
| `RabbitAdminService` | Los controladores no conocen RabbitMQ; toda la lógica está en el servicio. | `RabbitAdminService` |
| Validación de entrada | Nombre vacío o tipo inválido → 400 con mensaje claro. Borrar algo inexistente → 404. | DTO con `@Valid`; `ApiExceptionHandler` |
| Matriz de funcionalidades | 28 de 29 funcionalidades del alcance (96,6 %), 28 de 30 si se cuenta reportes (93,3 %). Cada fila con evidencia. | [MATRIZ-FUNCIONALIDADES.md](MATRIZ-FUNCIONALIDADES.md) |

## 2. Demo en vivo (orden sugerido, ~6 min)

Prerrequisitos: compose levantado (BFF, reservations, notify, RabbitMQ en clúster, Kafka), `ms-rabbit-admin` en ejecución aparte (puerto `8085`) y un usuario Admin con sesión iniciada.

1. **Clúster y colas (1 min):** UI de RabbitMQ (`:15672`) → Nodes (`rabbitmq-1` y `rabbitmq-2`) → Queues (`q.cmd.*` y sus `*.dlq`). Capturas `10` y `11`.
2. **Productor y consumidor (1 min):** desde la app, confirmar una reserva → mostrar el mensaje en la cola y el log de `notify`. Captura `12`.
3. **Mensaje inválido a la DLQ (1 min):** `POST /api/notify/demo/fail-email` → esperar un segundo → mensaje en `q.cmd.email.dlq` y log `NACK without requeue`. Captura `13`.
4. **Reintentos (1 min):** mostrar la salida de `mvn test -Dtest=ReliableDeliveryTest` con los logs `RETRY attempt=1/3 … 3/3` y `reasonKind=exhausted`. Captura `17`. Explicar que este camino se ejercita en los tests.
5. **Administrador (1,5 min):** `POST /api/queues` con nombre válido → 201; con `"name":""` → 400; `DELETE` de una cola inexistente → 404. Capturas `14`, `15` y `16`.
6. **Auditoría (30 s):** panel `/auditoria` y evento nuevo en `GET /api/audit/events` tras crear una reserva. Capturas `18` y `19`. Mostrar el evento nuevo solo si la prueba de punta a punta de Kafka ya se ejecutó con éxito; si no, enseñar el panel y los tests del listener.

Plan B si la infraestructura falla: usar las capturas de [evidencias/](evidencias/README.md) y mostrar los tests en verde de cada repositorio.

## 3. Límites que conviene declarar antes de que pregunten

- **Los reintentos ante error recuperable se demuestran con tests.** Hoy ningún sender real lanza `RecoverableDeliveryException`; el envío es un log.
- **`ms-rabbit-admin` no tiene autenticación y todavía no está en `compose.yml`.** Está pensado para la red interna y no se expone por el Gateway.
- **La publicación a RabbitMQ al confirmar una reserva es "best-effort":** si RabbitMQ falla, la reserva se confirma igual.
- **El Gateway en AWS** solo se da por cumplido (F20) cuando exista la prueba real con los curls (`20`). Hasta entonces la fila F20 está como parcial.
- **El consumidor Kafka de auditoría no deduplica por `eventId`.** Kafka entrega "al menos una vez", así que un reenvío podría duplicar un registro.

## 4. Preguntas probables

| Pregunta | Respuesta corta |
| --- | --- |
| ¿Por qué la DLQ y no reencolar? | Reencolar un mensaje que siempre falla lo reprocesa sin fin. El NACK sin requeue lo aparta en la DLQ y queda auditable. |
| ¿Cómo evitan procesar dos veces el mismo mensaje? | `ProcessedEventIds` reserva el `eventId`: un duplicado recibe ACK sin ejecutar el trabajo. |
| ¿Qué pasa si falla el envío del email? | Si el error es recuperable se reintenta con backoff y, al agotarse, va a la DLQ. Si no, va directo a la DLQ. Siempre queda en el log. |
| ¿Dónde se definen los nombres de colas? | En la configuración de cada servicio (`application.yml` o `application.properties`), nunca escritos en el código de negocio. |
| ¿El frontend habla directo con los microservicios? | No. Solo con el API Gateway o el BFF; el BFF valida el JWT y los roles. |
| ¿Cómo se valida el JWT? | El Gateway comprueba issuer, audience y scope; el BFF vuelve a validar issuer, audience, firma, expiración y roles. |

## 5. Antes de presentar

- [ ] Compose levantado y las capturas `10` a `20` subidas a `evidencias/`.
- [ ] Captura `05` reemplazada por una sin Bearer visible.
- [ ] Prueba del Gateway ejecutada y pegada (o F20 declarada como parcial).
- [ ] `mvn test` en verde en cada microservicio, con Docker encendido para `notify`.
- [ ] Enlaces de GitHub revisados y entrega en el AVA y por correo al docente.
