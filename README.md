# AndesStay — Docs

## Nombre

`andesstay-docs`

## Componente

Documentación del proyecto semestral AndesStay.

## Descripción

Arquitectura, identidad Azure AD, matriz de funcionalidades, evidencias (EP1/EP2/EP3) y guion de presentación.  
Un solo GitHub Project (Kanban) cubre este repo y el resto de componentes.

## Integrantes

- Cristian Monsalve
- Héctor Olivares
- Rolando Lillo

**Dueño del repositorio:** Cristian Monsalve

## Documentos

| Documento | Contenido |
| --- | --- |
| [IDENTIDAD.md](IDENTIDAD.md) | Tenant, App Registration, audience, scope, roles (sin secretos) |
| [ARQUITECTURA.md](ARQUITECTURA.md) | Diagrama JWT → Gateway → BFF → dominio (+ Rabbit/notify) |
| [MATRIZ-FUNCIONALIDADES.md](MATRIZ-FUNCIONALIDADES.md) | Alcance, estado, componente, evidencia (rúbrica sistematización) |
| [evidencias/](evidencias/README.md) | Checklist de capturas 200/401/403 y EP3 Rabbit |

Issue de cierre: [#3 Documentación y evidencia](https://github.com/cristianmonsalve14/andesstay-docs/issues/3).

## Tecnologías

- Markdown
- Diagramas (Mermaid)
- GitHub Projects (Kanban)

## Convención de ramas

- La rama `main` está protegida: **prohibido push directo**.
- Todo cambio entra por **Pull Request**.
- Se exige **al menos 1 reviewer** distinto al autor antes del merge.
