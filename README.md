# LIM COSMETIC — Ventas e inventario

Aplicación web para apoyar la operación de LIM COSMETIC: resumen del negocio, ventas, inventario, clientes, apartados e informes. La interfaz está construida con HTML, CSS y JavaScript, y utiliza Supabase para autenticación y datos.

> **Estado:** prototipo V1. Esta documentación describe la estructura observada en el repositorio. No sustituye las pruebas en producción ni la documentación del esquema de base de datos.

## Funciones principales
- **Inicio:** resumen de ventas del día, productos, apartados pendientes y existencias bajas.
- **Ventas:** selección de productos y registro de ventas.
- **Inventario:** consulta y gestión de productos y existencias.
- **Apartados:** pedidos apartados, abonos, saldos y estados.
- **Clientes:** listado y consulta de clientes.
- **Historial de ventas:** consulta de ventas registradas.
- **Reportes:** información de ventas y movimientos.
- **Configuración:** administración de empleados.
- **Factura PDF:** exportación de facturas mediante html2pdf.js.
- **Respaldo local:** descarga de un archivo JSON con los datos disponibles en el objeto de datos de la aplicación.

La disponibilidad de cada operación depende también de la configuración real de Supabase, sus tablas, políticas y funciones desplegadas.

## Roles y permisos

La aplicación reconoce dos roles en el perfil del usuario:

| Rol | Vistas disponibles en la interfaz |
| --- | --- |
| **Administrador** | Inicio, Ventas, Inventario, Apartados, Clientes, Historial de ventas, Reportes y Configuración |
| **Empleado** | **Solo Inicio, Ventas y Apartados** |

La interfaz oculta las opciones que no corresponden al empleado y la función de navegación lo devuelve a Inicio si intenta abrir una vista no permitida.

**Importante:** la visibilidad de botones y las validaciones JavaScript no bastan para proteger los datos. Supabase debe tener políticas Row Level Security (RLS) correctas y las funciones del servidor deben validar identidad y rol. La configuración completa de RLS y el código desplegado de las Edge Functions no se verificaron durante la documentación.

## Tecnologías
- HTML5, CSS y JavaScript del lado del cliente.
- Supabase para autenticación y acceso a datos.
- Supabase Edge Functions para operaciones administrativas de empleados.
- html2pdf.js para exportar facturas a PDF.
- Google Fonts para tipografías.

Las bibliotecas de Supabase y html2pdf.js se cargan desde CDN. No se identificó un proceso de compilación ni un archivo de dependencias de JavaScript en los archivos revisados.

## Estructura del proyecto

Rutas principales observadas:

- **index.html:** estructura principal de la aplicación y carga de módulos.
- **login.html:** pantalla de inicio de sesión.
- **css/styles.css:** estilos de la interfaz.
- **js/supabase-config.js:** URL y clave pública del proyecto Supabase.
- **js/auth.js:** verificación de sesión, perfil, rol y estado.
- **js/app.js:** navegación, vista Inicio, permisos visuales y utilidades.
- **js/productos.js:** consulta y operaciones de productos.
- **js/ventas.js:** flujo de ventas.
- **js/apartados.js:** flujo de apartados y sus detalles.
- **js/clientes.js:** funciones de clientes.
- **js/historial.js:** historial de ventas.
- **js/reportes.js:** reportes.
- **data/datos-demo.js:** datos de demostración o apoyo local.
- **recursos/logo.png:** logo de la aplicación.

El código del navegador también invoca las funciones de Supabase llamadas crear-empleado y eliminar-empleado. La implementación y despliegue de estas funciones deben revisarse en el proyecto Supabase; no se pueden deducir únicamente desde el frontend.

## Configuración y ejecución

### Requisitos
- Navegador moderno con JavaScript habilitado.
- Proyecto Supabase configurado para la aplicación.
- Usuario activo con rol administrador o empleado en la tabla perfiles.
- Servir los archivos desde HTTP/HTTPS; es preferible un servidor web local o el alojamiento del proyecto en vez de abrirlos directamente con el protocolo file.

### Pasos
1. Clona o descarga este repositorio.
2. Revisa js/supabase-config.js y confirma que la URL y la clave publicable corresponden al proyecto Supabase correcto.
3. En Supabase, verifica que existan las tablas, columnas, relaciones, políticas RLS y funciones que utiliza la aplicación.
4. Sirve los archivos con un servidor web local o publícalos en un alojamiento estático.
5. Abre login.html e inicia sesión con una cuenta activa.
6. Prueba los permisos de ambos roles antes de trabajar con datos reales.

No se identificó un comando de instalación o compilación en los archivos revisados; la aplicación se carga como archivos estáticos.

## Autenticación

En js/auth.js, la aplicación:
1. Consulta la sesión de Supabase.
2. Redirige a login.html si no hay sesión o se produce un error de autenticación.
3. Busca el perfil del usuario en la tabla perfiles, usando los campos id, nombre, rol y activo.
4. Cierra la sesión si el perfil no existe, está desactivado o tiene un rol no válido.
5. Expone el usuario y perfil actuales en window.usuarioActual y window.perfilActual.
6. Emite el evento usuarioAutenticado para que la interfaz configure el menú y abra Inicio.

La tabla perfiles y sus políticas de acceso deben configurarse en Supabase. Nunca guardes contraseñas en archivos del frontend.

## Datos y servicios externos

Las consultas revisadas hacen referencia a tablas como:
- perfiles
- productos
- ventas
- clientes
- apartados
- detalle_apartados
- movimientos_dinero
- detalle_ventas

Los campos exactos requeridos se encuentran en las consultas de cada módulo. El esquema completo, las restricciones y las políticas no están documentados aquí y deben contrastarse con la base de datos real.

El código invoca la función RPC eliminar_apartado para eliminar un apartado y devolver existencias, y funciones Edge para crear o eliminar empleados. Estas funciones deben validar permisos y datos del lado del servidor.

La URL de Supabase y una clave publicable del cliente pueden aparecer en el frontend. Una clave publicable no sustituye las políticas RLS. **Nunca publiques una clave service_role, contraseñas, tokens privados ni otros secretos en el navegador o en el repositorio.**

## Flujo general
1. El usuario inicia sesión en login.html.
2. La aplicación valida la sesión y carga el perfil.
3. La interfaz muestra las vistas permitidas para el rol.
4. Los módulos consultan o modifican datos de Supabase.
5. Las facturas PDF y las copias JSON locales se generan en el navegador cuando se ejecutan esas acciones.

## Pruebas recomendadas

Antes de cada despliegue:
- Iniciar sesión como empleado y comprobar que solo aparezcan Inicio, Ventas y Apartados.
- Intentar abrir directamente Inventario, Clientes, Historial, Reportes y Configuración como empleado; la aplicación debe volver a Inicio.
- Confirmar que RLS impida al empleado leer o modificar datos no autorizados incluso con solicitudes directas.
- Verificar que los empleados solo puedan consultar los apartados que les corresponden y que no puedan ejecutar operaciones administrativas.
- Probar ventas con stock suficiente, stock insuficiente y errores de red.
- Probar crear, abonar, vencer, entregar y eliminar apartados; verificar saldo y existencias.
- Contrastar los reportes con operaciones conocidas.
- Probar la generación de PDF y revisar los errores de la consola y los registros de Supabase.
- Utilizar datos ficticios en pruebas.

## Solución de problemas

| Síntoma | Qué revisar |
| --- | --- |
| Redirige al inicio de sesión | Sesión de Supabase, URL/clave pública, conexión y estado del perfil |
| No aparecen datos | Consola del navegador, nombres de tablas/columnas, conectividad y políticas RLS |
| No aparece una opción | Rol de la cuenta y permisos de la interfaz |
| No se crea o elimina un empleado | Respuesta y registros de las Edge Functions, permisos y variables de entorno del servidor |
| No se elimina un apartado | Disponibilidad de la RPC eliminar_apartado, permisos y políticas |
| No se genera el PDF | Carga del CDN de html2pdf.js y existencia del contenido de factura |

## Mantenimiento
- Actualiza esta documentación cuando cambien módulos, tablas o permisos.
- Revisa las dependencias externas y sus versiones de forma controlada.
- Añade pruebas automatizadas para permisos, ventas y apartados.
- Maneja los errores de red de forma consistente.
- Revisa RLS y las funciones del servidor al modificar roles o acciones sensibles.
- Realiza respaldos de la base de datos en Supabase. La descarga JSON del navegador no debe considerarse una copia completa de la base de datos.

## Licencia
No se identificó un archivo de licencia en los archivos revisados. Añade un archivo LICENSE si deseas definir los términos de uso y distribución.
