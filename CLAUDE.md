# CLAUDE.md — Reglas para construir el OS

## Qué es
Sistema operativo de negocio multiempresa para México: ERP + CRM + contabilidad + fiscal + nómina + operación en un solo lugar, con permisos por rol y **contabilidad y timbrado propios**.

Este repositorio construye el **núcleo universal: 402 funciones**, en 7 áreas y 46 departamentos. El catálogo vive en `docs/areas/`, un archivo por área. Los satélites de industria no se construyen ni se documentan aquí.

Fuente de verdad: la carpeta `docs/`. Si algo no está en los docs, pregunta antes de inventar.

| Área | Archivo del catálogo | Prefijo | Funciones |
|---|---|---|---|
| Finanzas | `docs/areas/01-finanzas.md` | `FIN` | 67 |
| Personas | `docs/areas/02-personas.md` | `PER` | 63 |
| Comercial | `docs/areas/03-comercial.md` | `COM` | 70 |
| Operaciones y Abastecimiento | `docs/areas/04-operaciones.md` | `OPE` | 76 |
| Legal y Riesgo | `docs/areas/05-legal-riesgo.md` | `LEG` | 50 |
| Dirección | `docs/areas/06-direccion.md` | `DIR` | 39 |
| Tecnología y Datos | `docs/areas/07-tecnologia-datos.md` | `TEC` | 37 |

Niveles: `E` Esencial (plan Basic, 118) · `P` Profesional (Pro, 281 acumuladas) · `A` Avanzado (Max, 402).

## Stack
Next.js (App Router) + TypeScript estricto · Supabase (PostgreSQL, Auth, Storage, Vault, RLS, pg_cron, Edge Functions) · Tailwind + shadcn/ui · Zod · React Hook Form · TanStack Table · Vitest · Playwright · PWA (Serwist) · Vercel · pnpm.

## Reglas de arquitectura (no negociables)
1. **Monolito modular.** Código por módulo en `src/modules/<modulo>`. Un módulo NO importa internals de otro; solo su `contracts/` (tipos y funciones públicas). Lo verifica `eslint-plugin-boundaries`.
2. **Un esquema de base de datos por módulo.** Un módulo solo escribe en su esquema. Nada de JOIN escribiendo cruzado.
3. **Comunicación por eventos.** Todo cambio relevante publica un evento en `plataforma.eventos_outbox` dentro de la MISMA transacción (patrón outbox). Ver `docs/05-catalogo-eventos.md`.
4. **Libro contable inmutable.** Solo el motor contable escribe en `contabilidad.*`. Prohibido UPDATE y DELETE en pólizas y partidas; los errores se corrigen con póliza de reversa.
5. **Multiempresa desde la primera migración.** Toda tabla de negocio tiene `empresa_id` y política RLS. Nunca se desactiva RLS, ni "un momento para probar".
6. **Datos sensibles.** CSD, e.firma, credenciales, CLABE: en Supabase Vault; en las tablas solo la referencia. Salarios: tabla con RLS restringida al rol Personas.
7. **Parámetros fiscales nunca en código.** UMA, salario mínimo, tablas ISR, subsidio, cuotas IMSS, tasas ISN: siempre desde `plataforma.parametros_fiscales` con vigencia por fecha.
8. **Idempotencia.** Los suscriptores de eventos registran `plataforma.eventos_procesados`; procesar dos veces no duplica.
9. **Bitácora.** Toda escritura de negocio deja rastro en `plataforma.bitacora`.
10. **Extensibilidad.** La orden de trabajo, la cotización, el activo y el empleado se diseñan genéricos: algún día un satélite de industria los extenderá agregando tablas en su propio esquema, sin modificar ninguna tabla del núcleo.

## Estructura
```
src/
  app/                  # rutas Next.js (UI) agrupadas por área
  platform/             # auth, tenant, permisos, eventos, bitácora, archivos, alertas
  engines/
    fiscal/             # CFDI, PAC, SAT, cálculos de impuestos
    contable/           # reglas evento→póliza, periodos, contabilidad electrónica
    nomina/             # ISR, subsidio, SDI, IMSS, INFONAVIT
    flujos/             # aprobaciones
    documentos/         # plantillas y PDF
  modules/
    comercial/ finanzas/ contabilidad/ operaciones/ personas/ legal/ direccion/ tecnologia/
      contracts/  domain/  application/  infra/  ui/  subscribers/  __tests__/
supabase/migrations/    # SQL versionado
docs/                   # especificación; docs/areas/ = catálogo de funciones
```

Los módulos de código corresponden uno a uno con las áreas del catálogo. Una función `FIN-xxx` se implementa en `src/modules/finanzas` (o `contabilidad`), nunca repartida entre módulos.

## Convenciones
- Dominio en español (factura, poliza, cobranza); SQL snake_case; TypeScript camelCase; UI en español de México.
- Dinero: `numeric(18,2)` en BD; en TS entero en centavos o decimal.js, nunca float.
- Fechas en UTC en BD; se muestran en America/Mexico_City.
- Validación de entrada con Zod en cada acción de servidor.
- Las acciones de servidor verifican permiso con `requirePermiso(recurso, accion)`.
- Un commit menciona la clave de la función: `feat(FIN-020): antigüedad de saldos`.

## Definición de terminado (cada tarea)
- Migración SQL con RLS + tipos regenerados.
- Lógica con pruebas (Vitest). Motores fiscal, contable y de nómina: pruebas con los casos dorados de `docs/12-pruebas-y-calidad.md`.
- Evento(s) publicados según el catálogo del doc 05.
- UI mínima funcional y protegida por permisos.
- `pnpm lint && pnpm typecheck && pnpm test` en verde.
- Estado de la tarea actualizado en `docs/11-plan-de-construccion.md`.

## Comandos
`pnpm dev` · `pnpm lint` · `pnpm typecheck` · `pnpm test` · `pnpm e2e` · `supabase db reset` · `supabase gen types typescript --local > src/platform/db/types.ts`

## Cómo trabajar
Una tarea a la vez del plan (`/siguiente-tarea`). Antes de codificar, resume qué vas a hacer y qué docs aplican.

La fase 0 del plan (plataforma y RLS · bus de eventos · libro contable · motores) es el cimiento del que dependen las siete áreas: no se recorta ni se reordena.

**Si una regla fiscal o contable es ambigua, detente y pregunta.** Esas reglas las valida un contador; no se deducen ni se aproximan.
