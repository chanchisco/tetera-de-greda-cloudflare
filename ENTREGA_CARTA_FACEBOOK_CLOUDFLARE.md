> Documento histórico del 5 de noviembre. Para el estado vigente y las correcciones publicadas, consultar ENTREGA_AJUSTES_FINALES_2026_11_06.md.

# Carta, Facebook y entrega independiente para Cloudflare

5 de noviembre de 2026. Cambios implementados localmente; no publicados desde este chat. Vercel, su repositorio, el dominio y los documentos de Firebase de producción no se modificaron.

## Cambios visibles

- Dos botones principales: **Comidas** y **Bebestibles**, antes del buscador. Las categorías Carnes, Del Mar, Entradas, Postres, etc. quedan exclusivamente para comida.
- Bebestibles muestra su carta agrupada por las subcategorías existentes. Se puede buscar por nombre, grupo y etiquetas, limpiar la búsqueda y volver a Comidas. Los controles son botones nativos, funcionan con teclado y anuncian la selección.
- Recomendados de la casa se oculta mientras se consulta Bebestibles. No se modificaron platos, bebidas, precios, imágenes, categorías guardadas ni el panel de administración.
- Facebook utiliza el enlace confirmado: https://web.facebook.com/RestobarLateteradegreda.
- En Inicio, Facebook aparece junto a Instagram en escritorio y debajo en celular, con el mismo formato. En el contacto de Inicio, Carta y Nosotros aparece también su botón junto al de Instagram, con ajuste de línea cuando hace falta.
- No se incrustan publicaciones, píxeles ni seguimiento de Facebook. Los enlaces abren la página elegida por el usuario.

## Dos entregas

1. `ENTREGA_CARTA_FACEBOOK_VERCEL_2026_11_05`: copia completa con las modificaciones visibles y la configuración actual para Vercel. Solo se sube al repositorio antiguo si se quiere actualizar también esa versión.
2. `ENTREGA_CLOUDFLARE_FINAL_2026_11_05`: copia completa para **otro repositorio**, con Cloudflare Workers + Static Assets. Contiene todos los cambios visibles, las optimizaciones anteriores y el servidor adaptado. Su versión ZIP contiene exactamente esa copia limpia.

No se deben mezclar las configuraciones de publicación. Para Cloudflare seguir `GUIA_CLOUDFLARE_PASO_A_PASO.md`, también incluida como README dentro de su entrega.

## Qué se adaptó para Cloudflare

El Worker publica `/api/contenido`, `/api/gestion`, `/api/publico`, `/api/imagen` y `/api/foto`. Conserva los manejadores de negocio probados: validación de origen, token revocado, roles, segundo factor cuando corresponda, límites, revisiones, historial y papelera. El navegador no recibe permisos de escritura adicionales.

La validación de acceso y operaciones de Authentication siguen usando Firebase Admin. Para Firestore se implementó un adaptador a su API REST oficial con lecturas, consultas, proyección de campos, cursores y transacciones atómicas; las transacciones canceladas por contención se reintentan como máximo tres veces. No se reemplazan transacciones por escrituras independientes.

Esto evita una incompatibilidad comprobada del cliente Firestore de Node: aun usando `preferRest`, parte de su código genera funciones dinámicamente, lo que no puede ejecutarse en Workers. La entrega final no utiliza ese cliente en el Worker. Vercel conserva su cliente original.

Las páginas estáticas las sirve el binding ASSETS. Las API públicas correctas pueden reutilizarse mediante Cache API de Cloudflare; las API privadas y los errores no se cachean. Se mantienen la caché pública de diez minutos, las imágenes inmutables y la copia local ante fallos. La caché de Cloudflare no se presenta como una caché global única.

La compilación de Cloudflare crea cabeceras equivalentes y configura el origen público a partir del sitio donde se abre, para que el QR use primero la URL provisional y luego `www.lateteradegreda.cl`. `APP_ORIGIN` sigue siendo un origen HTTPS exacto configurado en el servidor; no se aceptan comodines.

## Verificaciones

- **41 pruebas automatizadas** correctas, incluidas las protecciones anteriores y pruebas del adaptador REST, transacciones, cursores, tipos, caché y tamaño de solicitudes.
- **19 comprobaciones en workerd**, el runtime local de Cloudflare, con Auth y Firestore emulados: archivos estáticos, acceso anónimo rechazado, guardado con imagen, conflicto de revisiones, origen ajeno rechazado, editor sin Gestión, listado de cuentas, fuente privada, catálogo, imagen binaria, borrar/restaurar, cuenta desactivada, caché HIT, respaldo por páginas, actualización de estado/rol e historial.
- El entorno de prueba usó un proyecto `demo-tetera-cf`, tokens de Authentication emulado y un token de administrador exclusivo del emulador para el adaptador REST. Ese entorno y sus credenciales ficticias no se incluyen en las entregas. No se enviaron correos.
- Compilación de recursos y `wrangler deploy --dry-run` correctos. Paquete del Worker final aproximado: 1.388 KiB sin comprimir / 262 KiB gzip (puede variar ligeramente al reconstruir).
- Navegador local: cambio Comidas/Bebestibles, búsqueda y sin resultados, limpiar, volver con teclado, enlace correcto de Facebook y revisión de ambas resoluciones (1440px y 390px). Sin errores/advertencias capturados en la pestaña principal revisada. Se emplearon datos comerciales ficticios.

## Pendiente de activación real

Crear el repositorio y Worker en la cuenta del usuario, configurar los cinco valores de Firebase/origen, autorizar el dominio en Authentication y verificar acceso y métricas reales. La autenticación de cuenta de servicio contra Google y el límite de CPU del plan Free deben comprobarse en el despliegue real; las pruebas locales no certifican esos límites de producción.

Solo después se conecta el dominio desde la cuenta de Cloudflare/NIC Chile. No se prometen recursos ilimitados: cambiar de alojamiento no aumenta la cuota de Firestore. No se añadió un producto de pago ni se requiere activar Blaze como parte de esta entrega.

**Separar GitHub y el alojamiento no separa Firebase:** al configurar ambos sitios con `latetera`, comparten catálogo, perfiles y cuentas. Los cambios guardados desde cualquiera se reflejan en los mismos datos. Para probar escrituras completamente aisladas se necesita un Firebase de prueba con su propia configuración, además del repositorio nuevo.
