# Estado consolidado de sincronización LPS DUCA

Fecha de cierre: 2026-06-24
Repositorio: `rm022012-del/C-PROGRAMA-DE-ADUANA-LPS-duca-pc-app-fixed`

## 1. Propósito del documento

Este documento deja constancia del cierre operativo reportado para el entorno LPS DUCA después de la implementación del guardián de arranque, la revisión de sincronización de Google Drive y la identificación del pendiente estructural de infraestructura para control centralizado de correlativos.

La información operativa de auditoría se registra conforme al cierre proporcionado por el operador del sistema. No sustituye una auditoría independiente de base de datos ni una prueba formal en todas las estaciones de trabajo.

## 2. Estado implementado

### 2.1 Guardián de arranque versión 2

Se deja documentado que el guardián de arranque versión 2 fue instalado en la PC principal y publicado en el paquete operativo de Drive para distribución a las demás estaciones.

Funciones reportadas:

1. Reinicio controlado de Google Drive for Desktop cuando se detecten bloqueos de sincronización o lectura.
2. Protección del archivo de usuarios mediante respaldo de copia válida.
3. Restauración automática del archivo de usuarios si aparece dañado.
4. Contención de síntomas operativos reportados: ventana en blanco, cierre al entrar a una OT y mensaje de usuario inexistente.

### 2.2 Validación operativa observada

Durante la intervención se reportó una demostración práctica del incidente: la carpeta `OT-2620902393` quedó atascada en la caché de Google Drive y bloqueó la lectura. Al reiniciar Google Drive, la lectura respondió nuevamente.

Conclusión operativa: el mecanismo del guardián es consistente con el síntoma observado, porque actúa sobre el punto de bloqueo: Google Drive for Desktop y su caché local.

## 3. Hallazgos de auditoría reportados

Alcance reportado: 287 órdenes de trabajo revisadas.

Resultado consolidado reportado:

| Hallazgo | Resultado |
|---|---:|
| OT revisadas | 287 |
| Mezclas de carpetas | 0 |
| Archivos en conflicto | 0 |
| Trabajos duplicados confirmados | 0 |
| Casos a revisar | 2 |

Casos específicos:

| OT | Estado reportado | Acción sugerida |
|---|---|---|
| `OT-20902180` | PLUS MAKERS, vencida, sin XML y con número mal formado de origen | Decisión administrativa: conservar por trazabilidad o depurar con respaldo previo |
| `OT-2620902393` | Sin daño de datos; bloqueo asociado a caché de Drive | Mantener; caso normalizado tras reinicio de Drive |

Conclusión: no se reporta daño masivo de datos. El riesgo de duplicidad de correlativos existe como riesgo latente, no como desastre consumado en las 287 OT revisadas.

## 4. Instrucción operativa para cada PC

En cada estación de trabajo:

1. Abrir `Mi unidad\LPS_AGENCIA_ADUANAL\_GUARDIAN\`.
2. Ejecutar `INSTALAR_GUARDIAN.bat` con doble clic.
3. Priorizar instalación inicial en las PC de Stanley y Andrea.
4. Abrir el sistema mediante el acceso directo `LPS DUCA`.
5. Marcar la carpeta `LPS_AGENCIA_ADUANAL` en Google Drive como `Disponible sin conexión`.

Nota operativa: la disponibilidad sin conexión reduce la dependencia de lectura remota y disminuye bloqueos por caché o descarga tardía de archivos.

## 5. Límite de la solución actual

El guardián contiene y recupera bloqueos locales de Google Drive, pero no puede garantizar por sí solo que dos PC trabajando sin red tomen números de OT diferentes.

Para eliminar el riesgo de duplicados de correlativo se requiere un servicio centralizado de numeración, accesible por red, con control transaccional.

## 6. Pendiente estructural de infraestructura

Se requiere implementar la Fase 1: servidor central de correlativos.

Componentes mínimos:

1. VPS Linux pequeño y siempre disponible.
2. Base de datos PostgreSQL.
3. Secuencia transaccional de correlativos de OT.
4. Endpoint seguro para reservar número.
5. HTTPS obligatorio.
6. Autenticación por token o JWT.
7. Registro de auditoría por estación, usuario, fecha/hora e idempotency key.
8. Estrategia de rangos pre-reservados para contingencia sin Internet.
9. Respaldo automático del servidor.
10. Prueba de concurrencia con varias PC solicitando número simultáneamente.

## 7. Próxima acción recomendada

Enviar al programador el documento `docs/COTIZACION_VPS_FASE1.md` y solicitar el montaje del endpoint de prueba.

Una vez exista el endpoint, preparar y probar en banco el cliente local `CorrelativoClient`, sin modificar el núcleo de generación XML/PDF hasta aprobar la prueba de aceptación.

## 8. Criterios de aceptación de la Fase 1

La Fase 1 se considerará lista cuando:

1. Dos o más PC soliciten número de OT al mismo tiempo y reciban correlativos distintos.
2. El endpoint rechace solicitudes sin autenticación válida.
3. El sistema registre auditoría de cada reserva.
4. La base de datos mantenga integridad aunque una PC cierre el programa después de reservar número.
5. Exista respaldo automático del VPS.
6. Exista procedimiento documentado de reversa para volver temporalmente al método actual si la prueba falla.

## 9. Estado final

Estado operativo actual: contención implementada.

Riesgo residual: duplicidad de correlativos en escenarios de concurrencia o trabajo sin red.

Solución definitiva pendiente: servidor central de correlativos.
