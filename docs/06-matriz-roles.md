# 06 · Matriz de roles y permisos V1 (A3)

> **Roles del núcleo.** *Operaciones* coordina y despacha el trabajo; *Campo* es quien lo ejecuta y solo ve lo propio; *Colaborador* es cualquier empleado en su propio espacio (recibos, vacaciones, gastos). Un satélite de industria podría renombrarlos para su giro, pero no agrega roles nuevos.

Acciones: V ver · C crear · E editar · A aprobar · X exportar · — sin acceso · (p) solo propios.
El Administrador gestiona usuarios, roles, configuración e integraciones, sin acceso a contenido de negocio. El Propietario tiene todo.

| Recurso | Dirección | Tesorería | Contabilidad | Personas | Comercial | Operaciones | Campo | Colaborador |
|---|---|---|---|---|---|---|---|---|
| comercial.prospectos / oportunidades / actividades | V X | — | — | — | V C E X | V | — | — |
| comercial.clientes | V X | V | V | — | V C E X | V | — | — |
| comercial.clientes.credito | V A | V E | V | — | V | — | — | — |
| comercial.listas_precios | V A | — | — | — | V C E | V | — | — |
| comercial.cotizaciones / pedidos | V A X | V | V | — | V C E X | V | — | — |
| operaciones.ordenes / asignaciones | V X | V | V | — | V | V C E X | V (p) E (p) | — |
| operaciones.checklists / evidencias / incidencias | V | — | — | — | V | V C E | C (p) V (p) | — |
| operaciones.consumos | V X | V | V | — | — | V C E | C (p) | — |
| operaciones.proveedores | V | V C E | V | — | — | V C | — | — |
| operaciones.activos / mantenimientos / garantias | V X | V | V | — | — | V C E X | V (p) | — |
| finanzas.bancos / conciliacion | V X | V C E X | V E X | — | — | — | — | — |
| finanzas.cfdi_emitidos (timbrar/cancelar) | V | V | V C E | — | V C (desde pedido) | — | — | — |
| finanzas.cxc / cobranza | V X | V C E X | V | — | V | — | — | — |
| finanzas.cfdi_recibidos / cxp | V X | V C E | V C E X | — | — | V (clasificar op.) | — | — |
| finanzas.pagos_proveedor | V A | V C E | V | — | — | — | — | — |
| finanzas.gastos_empleado | V A | V E A | V E | — | C (p) | C (p) A | C (p) | C (p) |
| finanzas.arrendamientos / provisiones | V | V | V C E | — | — | — | — | — |
| finanzas.declaraciones / calendario | V | V | V C E X | V (isn, imss) | — | — | — | — |
| contabilidad.polizas manuales / cierre | V | — | V C A X | — | — | — | — | — |
| contabilidad.reportes (EF, balanza) | V X | V X | V X | — | — | — | — | — |
| analitica.rentabilidad / costeo | V X | V | V X | — | V (clientes) | V (unidades/rutas) | — | — |
| personas.empleados / expedientes / contratos | V | — | — | V C E X | — | V (su equipo) | V (p) | V (p) |
| personas.condiciones_salariales | — | — | — | V C E | — | — | — | — |
| personas.nomina / imss / isn | V A (totales) | V (dispersión) | V (póliza) | V C E X | — | — | V (p recibos) | V (p recibos) |
| personas.incidencias / vacaciones | V | — | — | V C E A | — | C A (su equipo) | C (p) | C (p) |
| legal.contratos / permisos / polizas_seguro / siniestros | V A X | V | V | — | V (clientes) | V C E | — | — |
| direccion.tablero / objetivos / alertas | V C E X | V (propias) | V (propias) | V (propias) | V (propias) | V (propias) | V (p) | V (p) |

Notas
- Dirección no ve salarios individuales ni datos médicos; ve totales de nómina.
- Montos de aprobación en `plataforma.reglas_aprobacion` (ej. pagos > umbral requieren Dirección).
- Alcance `(p)`: el rol Campo solo ve las órdenes asignadas a él, sus gastos y sus recibos. El Colaborador solo ve lo suyo.
- Toda acción de servidor usa `requirePermiso(recurso, accion)`; RLS protege por empresa y, para `(p)`, por usuario.
