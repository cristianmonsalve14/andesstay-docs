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
| Audience del JWT | `api://4cd6df9a-e2f7-4024-aea6-dd67c49709bc` |
| Scope | `access_as_user` |
| Scope completo | `api://4cd6df9a-e2f7-4024-aea6-dd67c49709bc/access_as_user` |

El frontend pide ese scope (MSAL + PKCE). El BFF debe validar **issuer**, **audience**, **firma**, **exp** y **roles**.

## App roles

Definidos en la App Registration (tipo **Users/Groups**, allowed member types).

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

Las contraseñas **no** van en este repo. Cada integrante las tiene por fuera.

## Qué valida cada capa

1. **Angular + MSAL:** login/logout, PKCE, guarda tokens, interceptor adjunta Bearer a llamadas a `apiUrl`.
2. **API Gateway:** CORS + JWT (issuer / audience). Sin token válido → 401.
3. **BFF:** misma validación JWT + roles. Sin rol → 403. Endpoint de evidencia: `/api/me`.
4. **Catálogo / reservas:** detrás del BFF; el navegador **no** los llama directo.

## Relación con issues

- Frontend: tenant y usuarios (`andesstay-frontend#1`), App Registration (`#2`), roles (`#3`).
- Docs: este archivo + diagrama en `ARQUITECTURA.md`.
