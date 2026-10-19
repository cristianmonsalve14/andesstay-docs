# Identidad — Azure AD (Entra ID)

Valores **públicos** que el frontend y el BFF necesitan para login y validación JWT.  
**No** se documentan contraseñas, client secrets ni certificados.

## Tenant

| Campo | Valor |
| --- | --- |
| Nombre | `proyecto1` |
| Dominio | `proyectomonsalve.onmicrosoft.com` |
| Tenant ID | `cb0b9f53-0ba7-4f09-8da2-c2f5ab4b73ee` |
| Issuer (v2) | `https://login.microsoftonline.com/cb0b9f53-0ba7-4f09-8da2-c2f5ab4b73ee/v2.0` |

## App Registration

| Campo | Valor |
| --- | --- |
| Nombre | AndesStay |
| Application (client) ID | `4cd6df9a-e2f7-4024-aea6-dd67c49709bc` |
| Plataforma | SPA |
| Redirect URI (local) | `http://localhost:4200` |
| Client secret | **Ninguno** (flujo PKCE; no hay secreto de aplicación) |
| Allow public client flows | No |
| Access token (versión) | 2 |

## API expuesta (audience / scope)

| Campo | Valor |
| --- | --- |
| Application ID URI | `api://4cd6df9a-e2f7-4024-aea6-dd67c49709bc` |
| Scope | `access_as_user` |
| Scope completo | `api://4cd6df9a-e2f7-4024-aea6-dd67c49709bc/access_as_user` |

El frontend pide ese scope (MSAL + PKCE).

**Audience en la práctica**

- El access token de MSAL suele traer `aud` = Client ID **sin** `api://` (`4cd6df9a-…`).
- El **BFF** acepta la audiencia configurada en su Resource Server (GUID y/o `api://…` según `application.yml`).
- El **API Gateway** JWT Authorizer es estricto: audience = Client ID **sin** `api://`, scope `access_as_user`. Si se configura `api://…` en el Gateway, el token válido de MSAL puede dar **401**.

## App roles

Definidos en la App Registration (tipo **Users/Groups**).

| Valor (`value`) | Display name |
| --- | --- |
| `Admin` | Admin |
| `Operador` | Operador |
| `Cliente` | Cliente |
| `Auditor` | Auditor |

Asignación 1:1 (Enterprise applications → AndesStay → Users and groups):

| Usuario | Rol |
| --- | --- |
| `andesstay.admin@proyectomonsalve.onmicrosoft.com` | Admin |
| `andesstay.operador@proyectomonsalve.onmicrosoft.com` | Operador |
| `andesstay.cliente@proyectomonsalve.onmicrosoft.com` | Cliente |
| `andesstay.auditor@proyectomonsalve.onmicrosoft.com` | Auditor |

Las contraseñas **no** van en este repo.

## Qué valida cada capa

1. **Angular + MSAL:** login/logout, PKCE, tokens, Bearer hacia `apiUrl`.
2. **API Gateway:** CORS + JWT (issuer / audience / scope). Sin token → **401**.
3. **BFF:** JWT + roles. Sin rol → **403**. Evidencia: `GET /api/me`.
4. **Catálogo / reservas / audit / notify:** detrás del BFF (o solo broker para notify); el navegador **no** los llama directo.

## Evidencia 200 / 401 / 403

Ver checklist y capturas en [evidencias/README.md](evidencias/README.md).

| Caso | Cómo | Código esperado |
| --- | --- | --- |
| Sin autenticación | Incógnito, `GET /api/me` sin Bearer | **401** |
| Token válido | Login + Dashboard `GET /api/me` | **200** + `roles` |
| Rol insuficiente | Cliente + `GET /api/catalog/units` | **403** |
| Logout | Cerrar sesión MSAL | Vuelta a `/login` |

## Relación con issues

- Frontend: tenant (`#1`), App Registration (`#2`), roles (`#3`), MSAL (`#4` / `#12`), guards (`#5` / `#13`).
- Docs: este archivo, [ARQUITECTURA.md](ARQUITECTURA.md), [MATRIZ-FUNCIONALIDADES.md](MATRIZ-FUNCIONALIDADES.md), issue [#3](https://github.com/cristianmonsalve14/andesstay-docs/issues/3).
