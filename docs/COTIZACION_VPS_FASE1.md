# Cotizacion VPS - Fase 1 servidor central de correlativos

Fecha de elaboracion: 2026-06-24
Repositorio: `rm022012-del/C-PROGRAMA-DE-ADUANA-LPS-duca-pc-app-fixed`

## 1. Objetivo de la Fase 1

Implementar un servidor central pequeno y siempre disponible para reservar correlativos de OT, evitando que dos PC tomen el mismo numero cuando trabajan de forma simultanea o cuando existen retrasos de sincronizacion en Google Drive.

El servidor no reemplaza Google Drive ni el programa actual. Su funcion principal es actuar como fuente central de numeracion.

## 2. Hallazgo operativo base

El guardian de arranque ayuda a contener bloqueos de Drive y restaurar archivos criticos, pero no puede garantizar por si solo numeracion unica cuando varias PC trabajan sin red o con sincronizacion tardia. Para eliminar ese riesgo se requiere un servicio central con base de datos y control transaccional.

## 3. Requerimiento minimo del VPS

Para la Fase 1 no se necesita un servidor potente. El servicio esperado maneja pocas solicitudes por minuto.

Configuracion minima recomendada:

- Linux Ubuntu LTS.
- 1 vCPU y 1-2 GB RAM como base aceptable.
- 2 vCPU y 4 GB RAM como opcion conservadora.
- PostgreSQL.
- API HTTPS.
- Firewall activo.
- Respaldos automaticos.
- Monitoreo basico de disponibilidad.

## 4. Proveedor recomendado con precio confirmado

### DigitalOcean

Opcion confirmada oficialmente al 2026-06-24:

- Droplet Basic 2 vCPU / 4 GiB RAM / 80 GiB SSD: USD 24.00 por mes.
- Droplet Basic 1 vCPU / 2 GiB RAM / 50 GiB SSD: USD 12.00 por mes.
- Backups: DigitalOcean indica backups porcentuales de 20% semanal o 30% diario sobre el costo del Droplet.

Estimacion operacional:

| Escenario | Compute | Backup semanal 20% | Total estimado mensual |
|---|---:|---:|---:|
| Minimo viable 1 vCPU / 2 GiB | USD 12.00 | USD 2.40 | USD 14.40 |
| Conservador 2 vCPU / 4 GiB | USD 24.00 | USD 4.80 | USD 28.80 |

No incluye dominio, impuestos, servicios administrados adicionales ni soporte externo del programador.

## 5. Vultr

Vultr puede ser una alternativa valida por ubicaciones cercanas a Centroamerica, como Miami o Ciudad de Mexico, pero no se deja precio cerrado en este documento porque no fue posible confirmar la tarifa oficial desde la pagina de precios al momento de elaborar este archivo. La consulta a la pagina oficial devolvio error de acceso. Debe verificarse directamente en la consola o pagina oficial de Vultr antes de contratar.

## 6. Comparacion contra ampliar Google Workspace

Ampliar Google Workspace puede mejorar almacenamiento, administracion o funciones de colaboracion, pero no resuelve por si mismo la asignacion atomica de correlativos. La numeracion unica requiere un punto central transaccional.

Google Workspace Business Standard informa 2 TB de almacenamiento por usuario en su pagina oficial de precios regional. Ese beneficio no sustituye una API central de correlativos.

## 7. Arquitectura de Fase 1

```text
PC con LPS DUCA
   |
   | POST /api/correlativos/ot/reservar
   v
API HTTPS en VPS
   |
   v
PostgreSQL con secuencia unica de OT
   |
   v
Respuesta: numero OT reservado
```

## 8. Endpoint requerido

### `POST /api/correlativos/ot/reservar`

Entrada sugerida:

```json
{
  "terminal_id": "PC-STANLEY-01",
  "usuario": "stanley",
  "idempotency_key": "uuid-generado-por-la-pc"
}
```

Respuesta sugerida:

```json
{
  "ok": true,
  "numero_ot": 2620902395,
  "reservado_en": "2026-06-24T00:00:00Z"
}
```

## 9. Controles obligatorios

- HTTPS obligatorio.
- JWT o token de servicio por PC.
- Idempotencia para evitar doble reserva si una PC reintenta por falla de red.
- Secuencia transaccional en PostgreSQL.
- Bitacora de cada reserva.
- Respaldo diario de base de datos.
- Monitoreo basico.
- Plan de reversa documentado.

## 10. Si una PC trabaja sin internet

Se recomienda implementar rangos pre-reservados pequenos por equipo, por ejemplo 5 o 10 correlativos. La PC solo debe consumir numeros previamente asignados por el servidor. Cuando regresa internet, reporta consumo y solicita nuevo bloque.

Esto evita que una PC invente numeros localmente.

## 11. Pruebas de aceptacion

1. Dos PC solicitan numero al mismo tiempo y reciben correlativos distintos.
2. Una PC reintenta la misma solicitud con igual `idempotency_key` y recibe el mismo numero, no uno nuevo.
3. El servidor registra usuario, PC, fecha y numero asignado.
4. Si PostgreSQL no responde, la API devuelve error controlado.
5. Si el token es invalido, la API rechaza la solicitud.
6. La base puede restaurarse desde respaldo.
7. El programa actual puede seguir trabajando con modo anterior si se activa reversa de emergencia.

## 12. Recomendacion ejecutiva

Para iniciar, contratar DigitalOcean Basic 1 vCPU / 2 GiB si se quiere minimizar costo o 2 vCPU / 4 GiB si se desea margen operativo. En ambos casos, activar backup semanal como minimo.

La decision recomendada para Fase 1 es DigitalOcean por precio oficial confirmado y suficiente capacidad para el volumen esperado.

## 13. Pendientes de infraestructura

- Elegir proveedor definitivo.
- Crear cuenta del proveedor.
- Registrar metodo de pago.
- Definir dominio o subdominio para HTTPS.
- Provisionar VPS.
- Instalar PostgreSQL y API.
- Sembrar secuencia inicial con el siguiente correlativo libre validado.
- Entregar endpoint al programador para integrar `CorrelativoClient`.

## 14. Fuentes consultadas

- DigitalOcean Droplet Pricing: https://www.digitalocean.com/pricing/droplets
- Google Workspace Pricing: https://workspace.google.com/pricing.html
- Vultr Pricing: https://www.vultr.com/pricing/ - pendiente de reconfirmacion directa por error de acceso al consultar.
