# Guía: auditoría de seguridad de mis proyectos (para portafolio)

Objetivo: convertir tus proyectos en evidencia de lo que piden las ofertas de seguridad:
auditoría de apps web y APIs, revisión de autenticación y permisos, informe con hallazgos priorizados y verificación de correcciones.

## Reglas
- Solo audita **tus propios proyectos** y entornos locales o de prueba.
- **CareFlow maneja datos de pacientes**: usa datos ficticios, nunca datos reales ni instancias de producción de instituciones.
- No publiques en GitHub secretos, URLs de producción con credenciales, ni detalles explotables de sistemas en uso.
- Cada hallazgo se documenta como "encontrado en mi proyecto de práctica, corregido en el commit X".

## Proyectos y ubicación local
| Proyecto | Ruta | Stack | Enfoque de auditoría |
|---|---|---|---|
| Matrix | `~/matrix` | Next.js, TypeScript, Supabase | Autenticación, permisos por rol, validación de entradas |
| CareFlow Nurse | `~/Documents/Desarrollo/Proyecto.integral` | React, Supabase (RLS) | RLS, roles de enfermería, exposición de datos de pacientes |
| Hoteles Centenario | `~/dev-local/hoteles-promo2` | Next.js estático | Cabeceras HTTP, configuración del servidor, dependencias |
| Shenzhen Metro Navigator | `~/dev-local/shenzhen-metro` | Python | Validación de entradas en el servidor web y la CLI |

## Fase 1: Preparación (1 semana)
1. Entorno local con datos ficticios. Nada de conexiones a Supabase de producción.
2. Inventario: lista de rutas, endpoints de API, roles y tablas. Guarda en `docs/seguridad/inventario.md`.
3. Herramientas gratuitas:
   - `npm audit` y `npm outdated` para dependencias.
   - OWASP ZAP (escaneo automatizado de la app local).
   - Burp Suite Community (interceptar peticiones y probar permisos a mano).
   - Semgrep (reglas estáticas para JavaScript/TypeScript).
   - Gitleaks (secretos en el historial del repo).

## Fase 2: Pruebas (2 semanas)
Por cada proyecto, revisa y anota evidencia:

**Autenticación**
- [ ] ¿Las rutas protegidas responden 401/403 sin sesión?
- [ ] ¿La sesión expira? ¿Las cookies tienen `HttpOnly`, `Secure` y `SameSite`?
- [ ] ¿Hay límite de intentos de login?

**Permisos (control de acceso)**
- [ ] Con un usuario de rol A, intenta leer o modificar un recurso de otro usuario (IDOR).
- [ ] En CareFlow: ¿una enfermera de una unidad puede ver pacientes de otra? Prueba las políticas RLS directamente.
- [ ] ¿El backend valida el rol, o solo la interfaz?

**Entradas y API**
- [ ] Envía valores inesperados (cadenas largas, caracteres especiales, tipos incorrectos) a cada endpoint.
- [ ] ¿Los errores muestran trazas o datos internos?
- [ ] ¿Hay validación del lado del servidor, no solo del formulario?

**Configuración**
- [ ] Cabeceras: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy`.
- [ ] Secretos: ¿hay claves en el código o en el repo? (`gitleaks detect`).
- [ ] Dependencias con vulnerabilidades: `npm audit`, anota severidad y si aplica al uso real.

## Fase 3: Informe (por proyecto)
Un documento corto, con esta estructura:

1. **Resumen:** alcance, fecha, herramientas y resultado general.
2. **Hallazgos priorizados:** cada uno con
   - ID, título y severidad (Crítica, Alta, Media, Baja).
   - Evidencia (captura o petición reproducible con datos ficticios).
   - Impacto: qué podría hacer un atacante y sobre qué datos.
   - Recomendación concreta.
   - Estado: pendiente / corregido (con enlace al commit).
3. **Verificación:** repetir la prueba después de corregir y anotar el resultado.

Guarda el informe como `docs/seguridad/informe-<proyecto>.md` en cada repo.

## Fase 4: Mostrarlo
- **README** de cada proyecto con una sección "Seguridad": qué se auditó, cuántos hallazgos, cuáles están corregidos y enlace al informe.
- **GitHub Actions** con `npm audit` y Semgrep en cada push. Así demuestras que la revisión es continua.
- **Frase para CV o perfil** (solo si ya lo hiciste):
  "Auditoría de seguridad de proyectos propios: pruebas de autenticación y control de acceso, revisión de APIs y cabeceras HTTP, informes con hallazgos priorizados y verificación de correcciones."

## Cómo usarlo para postular
- Las ofertas piden experiencia **comprobable**. Este portafolio lo es: informes, commits y pruebas reproducibles.
- Describe el trabajo como **proyecto personal**, no como experiencia laboral.
- Para ofertas de consultoría de seguridad, complementa con PortSwigger Web Security Academy (laboratorios gratuitos) y, si puedes, una certificación como eJPT o Security+.
