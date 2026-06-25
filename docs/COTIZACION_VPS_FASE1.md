# Cotización y plan de puesta en marcha - VPS Fase 1

Fecha de preparación: 2026-06-24
Proyecto: LPS DUCA - servidor central de correlativos
Repositorio: `rm022012-del/C-PROGRAMA-DE-ADUANA-LPS-duca-pc-app-fixed`

## 1. Objetivo ejecutivo

Implementar un servidor central de numeración para eliminar el riesgo de que dos estaciones de trabajo reserven el mismo número de OT.

La solución actual del guardián corrige bloqueos operativos de Google Drive y protege archivos locales críticos, pero no sustituye un control transaccional centralizado de correlativos.

## 2. Requerimiento mínimo

El servicio de correlativos no requiere un servidor grande. Requiere estabilidad, disponibilidad y baja latencia razonable hacia El Salvador.

Requerimiento técnico mínimo recomendado:

| Componente | Recomendación |
|---|---|
| Sistema operativo | Ubuntu LTS o Debian stable |
| CPU | 1 a 2 vCPU |
| RAM | 2 GB mínimo; 4 GB recomendado |
| Disco | SSD/NVMe, 25 GB o más |
| Base de datos | PostgreSQL |
| Exposición pública | API HTTPS |
| Seguridad | Firewall, SSH con llave, token/JWT, HTTPS obligatorio |
| Backups | Automáticos, mínimo semanal; ideal diario |

## 3. Proveedor recomendado con tarifa verificada

### Opción A - DigitalOcean

Fuente oficial consultada: `https://www.digitalocean.com/pricing/droplets`

DigitalOcean publica que sus Basic Droplets incluyen los siguientes planes relevantes:

| Plan | Recursos | Precio mensual publicado |
|---|---:|---:|
| Basic | 1 vCPU, 2 GiB RAM, 50 GiB SSD | USD 12/mes |
| Basic | 2 vCPU, 2 GiB RAM, 60 GiB SSD | USD 18/mes |
| Basic | 2 vCPU, 4 GiB RAM, 80 GiB SSD | USD 24/mes |

La misma fuente oficial indica que los backups de Droplets pueden contratarse como porcentaje del costo del Droplet: 20% semanal o 30% diario.

Cálculo estimado para producción mínima:

| Concepto | Cálculo | Total |
|---|---:|---:|
| Droplet 2 vCPU / 4 GiB | USD 24.00 | USD 24.00 |
| Backup semanal 20% | 24.00 x 0.20 | USD 4.80 |
| Subtotal técnico mensual | 24.00 + 4.80 | USD 28.80 |
| Dominio / DNS | Variable según proveedor | No incluido |

Costo técnico mensual estimado con DigitalOcean: **USD 28.80/mes**, sin incluir dominio ni impuestos aplicables.

Para una prueba de banco o piloto, puede arrancarse con el plan de 1 vCPU / 2 GiB por USD 12/mes, más backup semanal estimado en USD 2.40, total técnico aproximado: **USD 14.40/mes**.

## 4. Proveedor alternativo pendiente de confirmación directa

### Opción B - Vultr

Vultr es una alternativa razonable por cercanía de regiones como Miami y Ciudad de México; sin embargo, al preparar este documento no se pudo confirmar de forma directa y verificable la tarifa oficial desde la página de precios de Vultr, porque la consulta automatizada recibió bloqueo/error de acceso.

Por precisión documental, no se deja un precio de Vultr como tarifa confirmada. La recomendación es que el programador o el responsable de compra valide manualmente en la consola o sitio oficial de Vultr antes de contratar.

Criterio de selección si se usa Vultr:

1. Región: Miami o Ciudad de México, si está disponible en la cuenta.
2. Tamaño: 1-2 vCPU, 2-4 GB RAM.
3. Disco: SSD/NVMe.
4. Backup automático activado.
5. IPv4 pública y firewall habilitado.

## 5. Comparación ejecutiva contra ampliar Google Workspace

El problema de fondo no es almacenamiento ni permiso de edición de carpetas. El problema de fondo es concurrencia: dos PC pueden intentar tomar el mismo correlativo si no existe un punto central transaccional.

Por esa razón, aumentar el plan de Google Workspace por usuario no resuelve el control de correlativos. Un VPS central sí ataca directamente el riesgo.

Conclusión financiera: conviene pagar un servidor pequeño mensual para todo el sistema, no una mejora de plan por cada usuario o PC cuando el objetivo es numeración centralizada.

## 6. Arquitectura propuesta

```text
PC LPS DUCA
   |
   | POST /api/correlativos/ot/reservar
   v
API HTTPS en VPS
   |
   v
PostgreSQL
   |
   v
Secuencia transaccional de OT + bitácora de reserva
```

## 7. Endpoint mínimo requerido

### `POST /api/correlativos/ot/reservar`

Entrada sugerida:

```json
{
  "empresa": "LPS",
  "tipo": "OT",
  "usuario": "stanley",
  "equipo": "PC-STANLEY",
  "idempotency_key": "uuid-generado-por-la-pc"
}
```

Respuesta sugerida:

```json
{
  "ok": true,
  "correlativo": 2620902395,
  "serie": "OT",
  "reservado_en": "2026-06-24T20:00:00-06:00"
}
```

Reglas obligatorias:

1. La reserva debe ser transaccional.
2. La secuencia debe vivir en base de datos, no en archivo plano.
3. La `idempotency_key` debe evitar doble reserva si una PC reintenta por timeout.
4. Toda reserva debe quedar auditada.
5. El endpoint debe rechazar solicitudes sin token válido.

## 8. Base de datos sugerida

Tabla `ot_correlativo_reservas`:

```sql
CREATE TABLE ot_correlativo_reservas (
    id BIGSERIAL PRIMARY KEY,
    correlativo BIGINT NOT NULL UNIQUE,
    empresa TEXT NOT NULL,
    tipo TEXT NOT NULL DEFAULT 'OT',
    usuario TEXT NOT NULL,
    equipo TEXT NOT NULL,
    idempotency_key UUID NOT NULL UNIQUE,
    reservado_en TIMESTAMPTZ NOT NULL DEFAULT now(),
    usado BOOLEAN NOT NULL DEFAULT false,
    usado_en TIMESTAMPTZ NULL
);
```

Secuencia sugerida:

```sql
CREATE SEQUENCE ot_correlativo_seq START WITH 2620902395;
```

El número inicial debe confirmarse antes de producción contra el último correlativo real existente.

## 9. Rangos pre-reservados para contingencia sin Internet

Para evitar parálisis cuando una PC se quede sin Internet, puede implementarse un rango local pre-reservado por equipo.

Ejemplo operativo:

| Equipo | Rango reservado |
|---|---:|
| PC-STANLEY | 2620903000-2620903099 |
| PC-ANDREA | 2620903100-2620903199 |

Reglas:

1. Los rangos deben ser asignados por el servidor, no manualmente.
2. Cada PC debe reportar consumo del rango cuando recupere conexión.
3. El servidor debe impedir solapamientos.
4. El rango debe tener tamaño limitado para reducir impacto si una PC se daña.

## 10. Plan de implementación para el programador

### Fase 1.1 - Provisionamiento

1. Crear VPS.
2. Instalar Ubuntu LTS o Debian stable.
3. Configurar usuario administrativo sin acceso root por contraseña.
4. Activar firewall: permitir solo SSH, HTTP y HTTPS.
5. Instalar PostgreSQL.
6. Configurar backups del proveedor.

### Fase 1.2 - API

1. Crear endpoint `POST /api/correlativos/ot/reservar`.
2. Implementar autenticación por token/JWT.
3. Implementar idempotencia.
4. Implementar bitácora.
5. Exponer HTTPS con certificado válido.

### Fase 1.3 - Integración local

1. Preparar `CorrelativoClient` en el programa local.
2. Mantener fallback controlado, no silencioso.
3. Registrar errores de conexión.
4. No tocar generación XML/PDF hasta aprobar pruebas.

### Fase 1.4 - Prueba de aceptación

1. Tres PC solicitan número al mismo tiempo.
2. Deben recibirse tres números diferentes.
3. Se simula timeout y reintento con la misma `idempotency_key`.
4. El servidor debe devolver el mismo correlativo reservado, no crear otro.
5. Se corta Internet en una PC y se valida procedimiento de contingencia.
6. Se revisa bitácora y respaldo.

## 11. Entregables esperados

1. VPS configurado.
2. Repositorio o carpeta de código del endpoint.
3. Script SQL de base de datos.
4. Archivo `.env.example` sin secretos reales.
5. Manual de despliegue.
6. Manual de reversa.
7. Evidencia de prueba de concurrencia.
8. Evidencia de backup activo.

## 12. Decisión recomendada

Para avanzar con datos verificables, la opción recomendada es iniciar con DigitalOcean Basic 2 vCPU / 4 GiB por USD 24/mes, con backup semanal del 20%, para un estimado técnico de USD 28.80/mes antes de dominio e impuestos.

Si se prefiere validar costo mínimo, iniciar piloto con DigitalOcean Basic 1 vCPU / 2 GiB por USD 12/mes, backup semanal de USD 2.40, total técnico estimado de USD 14.40/mes antes de dominio e impuestos.

No se recomienda contratar un proveedor no verificado hasta confirmar manualmente precio, región, backup y método de pago.

## 13. Fuentes consultadas

- DigitalOcean Droplet Pricing: https://www.digitalocean.com/pricing/droplets
- Vultr Pricing: https://www.vultr.com/pricing/ - pendiente de reconfirmación directa por bloqueo/error de acceso al consultar.
