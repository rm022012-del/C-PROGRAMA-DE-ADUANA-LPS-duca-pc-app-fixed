# C-PROGRAMA-DE-ADUANA-LPS-duca-pc-app-fixed

Repositorio de trabajo para el programa de DUCA y, adicionalmente, para documentar el nuevo modulo propuesto de administracion de permisos de nube.

# Cloud Permission Manager - Google Drive

Programa propuesto para revisar y corregir permisos de carpetas en Google Drive / nube corporativa, con enfoque en evitar que carpetas autorizadas queden en modo solo lectura cuando deban permitir edicion.

## Objetivo

Automatizar la revision de permisos de carpetas de Google Drive y dejar a los usuarios autorizados con permiso de edicion, manteniendo trazabilidad de los cambios realizados.

## Alcance funcional

1. Conectarse a Google Drive mediante Google Drive API.
2. Revisar carpetas configuradas por la empresa.
3. Identificar permisos de solo lectura, por ejemplo Viewer o Commenter.
4. Cambiar permisos a Editor para usuarios o grupos autorizados.
5. Registrar auditoria de cada cambio.
6. Generar reporte de carpetas revisadas y permisos corregidos.
7. Ejecutarse de forma manual o programada.

## Aclaracion tecnica importante

Google Drive no funciona igual que una carpeta local de Windows. En Google Drive el control real depende de los permisos de acceso: Viewer, Commenter o Editor. Por eso el programa debe administrar permisos mediante la API de Google Drive.

Si los archivos tambien estan sincronizados en una PC con Google Drive for Desktop, puede existir una segunda capa local: el atributo de solo lectura de Windows. Esa revision local debe manejarse como modulo complementario, separado del control de permisos en la nube.

## Riesgo operativo

No se recomienda configurar la nube como "cualquiera puede editar" si eso implica usuarios externos o enlaces publicos. La configuracion recomendada es: usuarios autorizados de la empresa con permiso Editor.

## Estructura sugerida

- `docs/requerimiento-funcional.md`: alcance y criterios de aceptacion.
- `docs/diseno-tecnico.md`: arquitectura propuesta.
- `src/`: codigo fuente del programa.
- `logs/`: ubicacion sugerida para reportes locales de auditoria.

## Estado

Base inicial creada para levantar el requerimiento y comenzar desarrollo.
