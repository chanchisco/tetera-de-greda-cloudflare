# Publicar La Tetera de Greda en Cloudflare

Entrega del 5 de noviembre de 2026. Esta guía acompaña una copia completa preparada para otro repositorio. Vercel puede seguir funcionando mientras se prueba Cloudflare.

## 1. Qué vas a usar

Usaremos **Cloudflare Workers con Static Assets**. Es una aplicación que sirve las páginas y también las funciones privadas del administrador. No elegir la opción de publicar solamente HTML ni arrastrar solo `dist`: se perderían las funciones de guardado y Gestión.

El recorrido será:

**Repositorio nuevo de GitHub → Worker de Cloudflare → primero URL provisional → después www.lateteradegreda.cl.**

Se conserva Firebase para las cuentas y el contenido. **Si ambos alojamientos usan el proyecto `latetera`, comparten datos.** El repositorio nuevo protege la copia de archivos de Vercel; no crea una segunda base de datos.

## 2. Preparar el repositorio nuevo

1. Conserva tu respaldo habitual de la versión actual.
2. En GitHub crea un repositorio nuevo. Un nombre posible es `tetera-de-greda-cloudflare`; el nombre del repositorio puede ser otro.
3. Abre la carpeta `ENTREGA_CLOUDFLARE_FINAL_2026_11_05`. Si elegiste el ZIP, primero extráelo.
4. En el repositorio nuevo selecciona **Add file → Upload files**.
5. Sube **el contenido** de esa carpeta a la raíz del repositorio, incluidas las subcarpetas. No subas la carpeta exterior ni el ZIP cerrado.
6. Guarda con **Commit changes**.

En la raíz deben quedar, entre otros, `index.html`, `package.json`, `package-lock.json`, `wrangler.jsonc` y las carpetas `cloudflare`, `server` y `scripts`.

Usa esa entrega limpia. No arrastres toda tu carpeta de trabajo original: contiene respaldos y puede contener credenciales. La copia preparada excluye claves, exportaciones de datos, sesiones, `.firebase-private`, `node_modules` y capturas.

## 3. Conectar GitHub a Cloudflare

1. Entra a tu cuenta de Cloudflare y abre **Workers & Pages**.
2. Selecciona **Create application / Crear aplicación** y la importación desde **GitHub**.
3. Autoriza el repositorio nuevo y selecciónalo.
4. Nombre del Worker: **`tetera-de-greda-cf`**, igual que el valor `name` de `wrangler.jsonc`.
5. Si pide configuración de compilación, usa esta tabla:

| Campo | Valor |
|---|---|
| Directorio raíz | Raíz del repositorio, sin una subcarpeta |
| Comando de compilación | `npm run build` |
| Comando de despliegue | `npx wrangler deploy` |
| Versión Node para compilar | 22 |
| Configuración | `wrangler.jsonc` de la raíz |

La salida de recursos es `dist`, ya indicada en `wrangler.jsonc`; el servidor es `cloudflare/worker.js`. En Workers no hay que crear una carpeta Pages Functions.

6. Guarda y deja que termine el primer despliegue.
7. Cloudflare te mostrará una URL que termina en **`.workers.dev`**. Copia **esa URL real completa**, que será la dirección de prueba. No uses literalmente un ejemplo de esta guía.

En este primer momento las páginas pueden abrir, pero el administrador y los datos públicos todavía necesitan los valores del siguiente paso. Eso no significa que se haya perdido el contenido.

## 4. Configurar Firebase en el Worker

Dentro del Worker abre **Settings → Variables and Secrets**. Añade estos nombres exactamente:

| Nombre | Tipo sugerido | Qué poner |
|---|---|---|
| `FIREBASE_PROJECT_ID` | Texto | `latetera` |
| `FIREBASE_CLIENT_EMAIL` | Secret | El campo `client_email` de la cuenta de servicio que ya utilizas |
| `FIREBASE_PRIVATE_KEY` | Secret | El campo `private_key` de esa misma cuenta de servicio |
| `FIREBASE_WEB_API_KEY` | Secret | El valor público `apiKey` que está en `firebase-config.js` |
| `APP_ORIGIN` | Texto | La URL HTTPS real `.workers.dev` que acabas de obtener |

La clave privada puede pegarse con saltos de línea reales o con sus `\n`. No pegues todo el archivo JSON dentro de una sola variable: cada variable lleva su valor correspondiente.

`APP_ORIGIN` no lleva `/admin.html`, otra ruta ni barra al final. Debe coincidir con la dirección desde la que se abre el administrador. Más adelante se cambiará al dominio final.

No es necesario crear otra cuenta de servicio por esta migración: se usan los mismos campos que ya estaban configurados en Vercel. Las variables no se copian automáticamente entre las plataformas. No pongas estos valores en GitHub ni dentro de `wrangler.jsonc` o del archivo público `site-config.js`.

Guarda los valores y vuelve a desplegar el Worker para que se apliquen. La entrega incluye `keep_vars: true` para conservar las variables configuradas en el panel cuando se publique otra actualización desde GitHub.

## 5. Autorizar la dirección de prueba en Firebase

1. Abre Firebase, proyecto **latetera**.
2. Entra a **Authentication → Configuración → Dominios autorizados**.
3. Agrega el hostname exacto que Cloudflare te dio: solo el nombre que termina en `.workers.dev`, sin `https://` ni rutas.
4. Conserva los dominios de Vercel y Firebase que ya existían.

Si la clave web tiene restricciones de sitios en Google Cloud, añade también el hostname nuevo a sus sitios permitidos, conservando las restricciones y direcciones actuales. No hace falta quitar protecciones para migrar.

## 6. Probar antes de mover el dominio

Abre la dirección provisional con Ctrl + F5 y revisa:

1. Inicio, Nosotros y Carta.
2. Botones **Comidas / Bebestibles**, categorías, búsqueda y vuelta a Comidas.
3. Enlaces de Instagram y Facebook.
4. Fotos nuevas y antiguas.
5. Candado e ingreso con tu cuenta habitual.
6. Un cambio pequeño y reversible, su aparición en el administrador y su historial. Recuerda que usas el mismo Firebase y ese cambio también puede verse en Vercel.
7. Recuperación de acceso con tu propio correo, cuando quieras comprobar el envío real. Las pruebas locales no enviaron mensajes a nadie.

La caché pública conserva la optimización anterior: un cambio puede tardar aproximadamente diez minutos en verse al volver a abrir la carta. Si Firestore ya agotó su cuota diaria, hay que esperar su renovación; Cloudflare no restablece esa cuota.

En el Worker, revisar **Metrics / Métricas** y los errores después de estas pruebas. El plan Free tiene límites de solicitudes y CPU: la prueba local confirma funcionamiento, pero no mide la CPU aplicada por Cloudflare en producción. Si aparece un error de recursos o autenticación, revisarlo antes de conectar el dominio; no cambiar a un plan pagado como solución automática.

No hace falta volver a publicar reglas de Firebase por esta migración ni cambiar contraseñas o perfiles.

## 7. Añadir lateteradegreda.cl a Cloudflare

Este paso se hace cuando el Worker provisional funciona y tienes acceso a la administración del dominio.

1. En Cloudflare agrega **`lateteradegreda.cl`** como dominio/zona y elige el plan Free.
2. Revisa los registros DNS que detecte. Conserva los del correo y otros servicios que utilice el negocio, especialmente MX y los TXT de SPF/DKIM/DMARC si existen.
3. Cloudflare te asignará dos **nameservers / servidores de nombre**, normalmente con nombres terminados en `.ns.cloudflare.com`. Copia los que asigne tu cuenta; no se pueden anticipar en esta guía.
4. En la cuenta que administra el dominio en NIC Chile, abre `lateteradegreda.cl` y su sección **Servidores de nombre / DNS**.
5. Reemplaza los servidores de nombre por esos dos de Cloudflare y guarda. Antes conserva un registro de la configuración anterior.
6. Espera a que Cloudflare marque la zona como **Active / Activa**. La propagación DNS puede tardar; no borres Vercel mientras se completa.

Cambiar nameservers significa que Cloudflare administrará los registros DNS. No es necesario transferir la propiedad del dominio ni volver a comprarlo para este procedimiento. El registro/renovación del dominio es independiente del alojamiento.

## 8. Conectar www.lateteradegreda.cl al Worker

1. Abre el Worker **tetera-de-greda-cf**.
2. Ve a **Settings → Domains & Routes → Add → Custom Domain**.
3. Escribe **`www.lateteradegreda.cl`** y confirma la asociación.
4. Deja que Cloudflare prepare su registro DNS y certificado HTTPS. Si ya existe un registro para ese hostname, conserva su valor anterior y resuelve el conflicto como parte de la asociación, sin alterar registros ajenos como el correo.
5. Cuando esté activo, cambia `APP_ORIGIN` del Worker a:

   `https://www.lateteradegreda.cl`

6. En Firebase Authentication agrega **`www.lateteradegreda.cl`** a los dominios autorizados. Conserva los anteriores.
7. Vuelve a desplegar el Worker después de cambiar la variable y realiza el ingreso desde **la dirección con www**.
8. Comprueba un guardado, fotos y las páginas públicas otra vez.

El QR toma la dirección desde la que abres este panel en la compilación de Cloudflare. Genera el QR definitivo entrando por `https://www.lateteradegreda.cl`, no por la dirección provisional, antes de imprimir para las mesas.

## 9. Dirección sin www, si quieres que también funcione

La dirección principal elegida es **www.lateteradegreda.cl**. Para que `lateteradegreda.cl` sin www envíe a esa dirección, configura en la zona una **Redirect Rule** de Cloudflare, usando su plantilla **Redirect from root to WWW** y conservando ruta y parámetros.

El hostname de origen debe tener un registro DNS con proxy activo para que la regla reciba las solicitudes. La plantilla y su documentación oficial indican la configuración de ese registro. Esta redirección se configura en Cloudflare; no se incluyó una regla absoluta inválida dentro de `_redirects`.

## 10. Mantener Vercel como respaldo

Conserva el repositorio y proyecto de Vercel actuales. El repositorio nuevo publica únicamente el Worker nuevo. Si usas el mismo Firebase, volver a Vercel recupera ese alojamiento, pero no revierte cambios hechos en los datos compartidos.

No se publicaron recursos, modificaron DNS ni configuraron secretos reales desde este chat. Esos pasos pertenecen a tu cuenta y se completan después de subir esta entrega.

## Documentación consultada

- [Workers con recursos estáticos](https://developers.cloudflare.com/workers/static-assets/)
- [Conectar un repositorio a Workers](https://developers.cloudflare.com/workers/ci-cd/builds/)
- [Variables secretas](https://developers.cloudflare.com/workers/configuration/secrets/)
- [Dominios propios en Workers](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/)
- [Redirección de raíz a www](https://developers.cloudflare.com/rules/url-forwarding/examples/redirect-root-to-www/)
- [Límites de Workers](https://developers.cloudflare.com/workers/platform/limits/)
- [Servidores DNS en NIC Chile](https://www.nic.cl/ayuda/faq/ins-05.html)
