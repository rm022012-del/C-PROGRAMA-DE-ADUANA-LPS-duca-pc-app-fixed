# Diseno tecnico - Cloud Permission Manager Google Drive

## 1. Arquitectura propuesta

El sistema se divide en cuatro componentes principales:

1. Modulo de autenticacion.
2. Modulo de lectura de permisos.
3. Modulo de correccion de permisos.
4. Modulo de auditoria y reportes.

## 2. Flujo operativo

```text
Inicio
  |
  v
Cargar configuracion de carpetas autorizadas
  |
  v
Autenticar contra Google Drive API
  |
  v
Leer permisos actuales de cada carpeta
  |
  v
Comparar contra usuarios/grupos autorizados
  |
  v
Si permiso es Viewer o Commenter, cambiar a Editor
  |
  v
Registrar resultado en bitacora
  |
  v
Generar reporte final
Fin
```

## 3. Tecnologia sugerida

- Lenguaje: Python 3.11 o superior.
- API: Google Drive API v3.
- Autenticacion: OAuth 2.0 o cuenta de servicio, segun politica de Google Workspace.
- Configuracion: archivo JSON.
- Auditoria: CSV, JSONL o base de datos liviana.
- Ejecucion: manual, tarea programada de Windows o servicio interno.

## 4. Entidades principales

### Carpeta configurada

- `folder_id`: identificador de carpeta en Google Drive.
- `name`: nombre descriptivo.
- `authorized_principals`: usuarios o grupos autorizados.
- `target_role`: rol objetivo, normalmente `writer`.
- `dry_run`: si esta activo, no aplica cambios reales.

### Registro de auditoria

- Fecha y hora.
- ID de carpeta.
- Nombre de carpeta.
- Usuario o grupo afectado.
- Rol anterior.
- Rol nuevo.
- Resultado.
- Mensaje de error, si existe.

## 5. Roles de Google Drive API

En Google Drive API el rol de edicion normalmente se representa como `writer`. Los roles de lectura o menor privilegio pueden incluir `reader` o `commenter`.

## 6. Seguridad

- No usar permisos publicos por defecto.
- No guardar credenciales dentro del repositorio.
- Usar variables de entorno o secretos administrados.
- Activar modo simulacion antes de aplicar cambios reales.
- Mantener bitacora de todo cambio.

## 7. Modulo local opcional para Windows

Si se requiere revisar atributos locales en PC sincronizadas con Google Drive for Desktop, se puede agregar un modulo separado para validar si los archivos tienen atributo de solo lectura en Windows. Este modulo no sustituye los permisos reales de Google Drive.

## 8. Pendientes tecnicos

- Definir si se usara OAuth de usuario o cuenta de servicio.
- Confirmar si la cuenta tiene permisos de propietario o administrador sobre las carpetas.
- Definir lista oficial de usuarios o grupos autorizados.
- Definir si se aplicara a unidades compartidas de Google Workspace.
