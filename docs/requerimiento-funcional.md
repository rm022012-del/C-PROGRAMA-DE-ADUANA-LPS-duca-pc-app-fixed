# Requerimiento funcional - Cloud Permission Manager Google Drive

## 1. Proposito

Implementar un modulo que revise permisos de carpetas de Google Drive y corrija accesos de solo lectura cuando los usuarios autorizados deban tener permiso de edicion.

## 2. Problema operativo

En el uso diario de carpetas compartidas en la nube, algunos usuarios o carpetas pueden quedar configurados con permisos de lectura. Esto impide editar documentos, modificar archivos o trabajar sobre carpetas compartidas desde equipos autorizados.

## 3. Resultado esperado

El sistema debe validar permisos de carpetas configuradas y dejar con rol Editor a los usuarios, grupos o unidades autorizadas por la administracion.

## 4. Alcance incluido

- Autenticacion contra Google Drive API.
- Lectura de permisos actuales de carpetas.
- Identificacion de roles Viewer y Commenter.
- Correccion a rol Editor cuando aplique.
- Bitacora de cambios.
- Reporte de ejecucion.
- Ejecucion manual y posibilidad de tarea programada.

## 5. Alcance excluido inicialmente

- Otorgar permisos de edicion a usuarios externos no autorizados.
- Hacer publica una carpeta para cualquier usuario de internet.
- Eliminar controles de seguridad de Google Workspace.
- Modificar archivos sin respaldo de auditoria.

## 6. Reglas de negocio

1. Solo se deben modificar carpetas autorizadas en la configuracion del sistema.
2. Solo se deben elevar permisos de usuarios o grupos incluidos en la lista autorizada.
3. Todo cambio debe quedar registrado con fecha, hora, carpeta, permiso anterior, permiso nuevo y usuario afectado.
4. Si el sistema no tiene permiso para modificar una carpeta, debe registrar el error y continuar con las demas carpetas.
5. No debe borrar permisos existentes sin instruccion expresa.

## 7. Criterios de aceptacion

- El sistema lista las carpetas configuradas.
- El sistema detecta permisos Viewer o Commenter.
- El sistema cambia a Editor los permisos autorizados.
- El sistema genera una bitacora de auditoria.
- El sistema informa errores sin detener todo el proceso.
- El sistema no cambia permisos de carpetas no configuradas.

## 8. Datos minimos de configuracion

- ID de carpeta de Google Drive.
- Nombre descriptivo de carpeta.
- Usuarios o grupos autorizados.
- Rol objetivo: Editor.
- Modo de ejecucion: simulacion o aplicacion real.

## 9. Modo simulacion

Antes de aplicar cambios reales, el sistema debe poder ejecutarse en modo simulacion para mostrar que permisos cambiaria sin modificar Google Drive.

## 10. Control de riesgo

La opcion recomendada es permitir edicion solo a usuarios autorizados de la empresa, no a cualquier persona con enlace publico.
