# Specs — Fuente de Verdad del Proyecto ASDD

Este directorio contiene las especificaciones técnicas de cada funcionalidad. Son la fuente de verdad para todos los agentes de desarrollo.

## Ciclo de Vida

```
DRAFT → APPROVED → IN_PROGRESS → IMPLEMENTED → DEPRECATED
```

| Estado | Quién | Condición |
|--------|-------|-----------|
| `DRAFT` | spec-generator | Spec generada, pendiente de revisión humana |
| `APPROVED` | Usuario / Tech Lead | Revisada y aprobada — verde para implementar |
| `IN_PROGRESS` | orchestrator | Implementación en curso |
| `IMPLEMENTED` | orchestrator | Código + tests + QA completos |
| `DEPRECATED` | Usuario | Descartada o reemplazada por otra spec |

> **Regla:** Sin `status: APPROVED` en el frontmatter → ningún agente implementa código.

## Convención de Nombres

```
.github/specs/<nombre-feature-en-kebab-case>.spec.md
```

## Índice de Specs

| ID | Feature | Funcionalidades | Archivo | Estado | Fecha |
|----|---------|----------------|---------|--------|-------|
| SPEC-001 | product-list | F1, F2, F3 | [product-list.spec.md](product-list.spec.md) | APPROVED | 2026-05-10 |
| SPEC-002 | add-product | F4 | [add-product.spec.md](add-product.spec.md) | APPROVED | 2026-05-10 |
| SPEC-003 | edit-product | F5 | [edit-product.spec.md](edit-product.spec.md) | APPROVED | 2026-05-10 |
| SPEC-004 | delete-product | F6 | [delete-product.spec.md](delete-product.spec.md) | APPROVED | 2026-05-10 |

> Actualizar esta tabla cada vez que se crea o cambia el estado de una spec.

## Plan de Specs — Prueba Técnica (F1–F6)

| Orden | Feature Sugerido | Funcionalidades | Comando |
|-------|-----------------|----------------|---------|
| 1 | `product-list` | F1 + F2 + F3 | `/generate-spec product-list` |
| 2 | `add-product` | F4 | `/generate-spec add-product` |
| 3 | `edit-product` | F5 | `/generate-spec edit-product` |
| 4 | `delete-product` | F6 | `/generate-spec delete-product` |

> **Sugerencia**: F1, F2 y F3 son parte de la misma pantalla (`ProductListScreen`) — conviene agruparlas en una sola spec.

## Requerimientos pendientes de spec

Los siguientes requerimientos están en `.github/requirements/` listos para convertirse en spec:

| Requerimiento | Archivo | Acción |
|---------------|---------|--------|
| Prueba Técnica — Productos Financieros (F1–F6) | `.github/requirements/financial-products.md` | `/generate-spec product-list` |

## Cómo crear una spec nueva

**Opción 1 — Desde un requerimiento existente:**
```
/generate-spec financial-products
```

**Opción 2 — Feature específico:**
```
/generate-spec product-list
> Funcionalidades F1 (listado), F2 (búsqueda) y F3 (contador) de la prueba técnica
```

**Opción 3 — Orquestación completa (spec → implementación → tests → QA):**
```
/asdd-orchestrate
> Feature: product-list
```

## Frontmatter requerido en toda spec

```yaml
---
id: SPEC-001
status: DRAFT
feature: nombre-del-feature
created: YYYY-MM-DD
updated: YYYY-MM-DD
author: spec-generator
version: "1.0"
related-specs: []
---
```

## Template

Ver `.github/skills/generate-spec/spec-template.md`
