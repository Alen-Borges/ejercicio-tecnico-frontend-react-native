# Copilot Instructions

## ASDD Workflow (Agent Spec Software Development)

Este repositorio sigue el flujo **ASDD**: toda funcionalidad nueva se ejecuta en fases orquestadas por agentes especializados.

```
[Orchestrator] → [Spec Generator] → [Frontend Developer] → [Test Engineer Frontend] → [QA Agent]
```

> **Nota**: Este es un proyecto **solo frontend**. No hay backend que implementar — el backend corre localmente en `http://localhost:3002`.

### Fases del flujo ASDD

1. **Spec**: El agente `spec-generator` genera la spec en `.github/specs/<feature>.spec.md`.
2. **Implementación**: `frontend-developer` implementa en React Native + TypeScript.
3. **Tests**: `test-engineer-frontend` genera pruebas con Jest + RNTL (coverage ≥ 70%).
4. **QA**: `qa-agent` genera Gherkin, riesgos y colección Postman para el backend local.

### Skills disponibles (slash commands):

- `/asdd-orchestrate` — orquesta el flujo completo ASDD
- `/generate-spec` — genera spec técnica en `.github/specs/`
- `/implement-frontend` — implementa feature completo en React Native + TypeScript
- `/unit-testing` — genera suite de tests (Jest + React Native Testing Library)

---

## Stack del Proyecto

| Tecnología | Versión | Uso |
|-----------|---------|-----|
| React Native | 0.73+ | Framework mobile |
| TypeScript | 4.8+ | Tipado estático |
| React | 18+ | UI |
| React Navigation | v6 | Navegación Stack |
| Jest | latest | Testing runner |
| @testing-library/react-native | latest | Testing de componentes |

**Backend local**: Node.js en `http://localhost:3002` (no implementar, solo consumir).

---

## Mapa de Archivos e Instrucciones

### Instrucciones Path-Scoped

| Scope | Instrucción | Se aplica a |
|---|---|---|
| **Frontend** | `.github/instructions/frontend.instructions.md` | `src/**/*.{ts,tsx}` |
| **Tests** | `.github/instructions/tests.instructions.md` | `src/__tests__/**/*.{ts,tsx}` |

### Lineamientos y Contexto

| Documento | Ruta |
|---|---|
| Lineamientos de Desarrollo | `.github/docs/lineamientos/dev-guidelines.md` |
| Lineamientos QA | `.github/docs/lineamientos/qa-guidelines.md` |

---

## Reglas de Oro

> Principio rector: cada acción debe ser relevante, transparente y alineada con las instrucciones explícitas.

1. **React Native + TypeScript**: Componentes funcionales, hooks, `StyleSheet.create()`. Sin frameworks UI externos.
2. **No implementation without a spec**: Consultar siempre `.github/specs/` antes de codificar.
3. **Clean Code + SOLID**: Principios de diseño en todas las capas. Sin código redundante.
4. **Coverage ≥ 70%**: Requerido por la prueba técnica — gate bloqueante.
5. **Manejo de errores visual**: Siempre mostrar feedback al usuario en caso de error.

---

## Diccionario de Dominio (React Native Context)

| Término | Definición | Contexto Técnico |
|---------|-----------|------------------|
| **FinancialProduct** | Entidad principal del dominio | Interface TypeScript en `src/models/` |
| **Service** | Capa de llamadas HTTP | Archivo en `src/services/` — solo fetch, sin estado |
| **Hook** | Lógica de estado y efectos | Archivo en `src/hooks/` — consume services |
| **Component** | UI reutilizable | Directorio en `src/components/` — props tipadas |
| **Screen** | Vista completa / página | Directorio en `src/screens/` — composición |
| **Navigator** | Sistema de rutas | `src/navigation/AppNavigator.tsx` |

---

## Funcionalidades de la Prueba Técnica

| ID | Feature | Componentes / Screens | API |
|----|---------|----------------------|-----|
| F1 | Listado de productos | `ProductListScreen`, `ProductCard` | `GET /bp/products` |
| F2 | Búsqueda por texto | `ProductListScreen` (filtro local) | — |
| F3 | Contador de registros | `ProductListScreen` | — |
| F4 | Agregar producto | `AddProductScreen`, `ProductForm` | `POST /bp/products`, `GET /bp/products/verification/:id` |
| F5 | Editar producto | `EditProductScreen`, `ProductForm` | `PUT /bp/products/:id` |
| F6 | Eliminar producto | `DeleteModal` | `DELETE /bp/products/:id` |

---

## Project Overview

> Ver `README.md` en la raíz del proyecto para instrucciones de setup y ejecución.
