# Notas: OWASP Juice Shop (local, 127.0.0.1:3000)

Entorno: Docker, imagen bkimminich/juice-shop, puerto ligado a 127.0.0.1. Solo datos de laboratorio.
Fecha de inicio: 2026-10-19

## Estado de los ejercicios

| # | Tema | Estado | Resultado |
|---|---|---|---|
| 1 | Inyección SQL en login | Confirmado | Entrar como admin con un correo manipulado; la comilla simple muestra error de servidor con traza HTML |
| 2 | XSS / inyección HTML reflejada en buscador | Observado, pendiente de confirmar | El encabezado muestra "texto_de_prueba" sin las etiquetas `<b>`. Falta confirmar en el DOM con las herramientas del navegador |
| 3 | IDOR en carrito (`/rest/basket/:id`) | Confirmado | usuarioA lee el carrito de usuarioB (200); sin token responde 401 |

## 1. Inyección SQL en login (confirmado)
- Prueba 1 (control): correo válido con contraseña incorrecta → 401.
- Prueba 2 (comilla simple en el correo) → página de error HTML con traza del servidor. Divulgación de información además de la inyección.
- Prueba 3 (correo: `' OR 1=1--`, contraseña cualquiera) → 200 con token válido y el correo `admin@juice-sh.op`. Entró como administrador sin contraseña.
- Por qué funciona: el correo se concatena en la consulta SQL; el `--` comenta el resto, incluida la comprobación de la contraseña.
- Impacto: acceso como administrador sin credenciales.
- Corrección propuesta: consultas parametrizadas (placeholders), nunca concatenar la entrada; respuestas de error genéricas sin trazas en producción; límite de intentos de login.
- Verificación pendiente: repetir las tres pruebas con la corrección aplicada y confirmar 401 y ausencia de trazas.

## 2. XSS / inyección HTML reflejada en buscador
- Entrada de prueba: `#/search?q=%3Cb%3Etexto_de_prueba%3C%2Fb%3E`
- Resultado observado: el encabezado "Search Results - texto_de_prueba" aparece sin las etiquetas.
- Confirmación en DOM: pendiente (¿existe un `<b>` dentro del encabezado?).
- Por qué ocurre: el texto de la búsqueda se inserta como HTML sin escapar.
- Corrección propuesta: codificar la salida; no usar `innerHTML` ni `dangerouslySetInnerHTML`; CSP en cabeceras.
- Verificación: repetir la prueba y confirmar que aparece el texto literal.

## 3. IDOR en carrito (confirmado)
- Usuarios de prueba: usuarioA (id 25, carrito 6), usuarioB (id 26, carrito 7). Creados en el laboratorio.
- Petición propia: GET /rest/basket/6 con token de usuarioA → 200
- Petición con carrito ajeno: GET /rest/basket/7 con token de usuarioA → 200. La respuesta trae "UserId": 26, que no es el usuario de la sesión.
- Sin token: GET /rest/basket/7 → 401.
- Impacto: un usuario autenticado puede leer el carrito de otro usuario cambiando el identificador.
- Corrección propuesta: en el servidor, comparar el UserId del carrito con el usuario de la sesión; responder 403 si no coinciden. No confiar en el identificador que llega en la URL.
- Verificación pendiente: repetir la petición después de corregir y confirmar 403.
- Nota: el token no se guarda en este archivo.

## 4. Inyección SQL en búsqueda de productos (API) (confirmado)
- Endpoint: GET /rest/products/search?q=
- Prueba 1 (control): q=apple → 3 productos.
- Prueba 2: q=' → 200 sin mensaje de error visible en la respuesta; no se reportó traza en esta prueba.
- Prueba 3: q=')) OR 1=1-- → 56 productos (todo el catálogo).
- Por qué funciona: el texto de búsqueda se concatena en la consulta SQL; la condición `OR 1=1` vuelve verdadera la cláusula WHERE y el `--` comenta el resto.
- Impacto: lectura de datos más allá de lo que la búsqueda debería devolver.
- Corrección propuesta: consultas parametrizadas en la búsqueda; validar longitud y caracteres; no exponer errores de base de datos.
- Verificación pendiente: repetir la prueba 3 con la corrección y confirmar que devuelve 3 o 0 resultados, no 56.
