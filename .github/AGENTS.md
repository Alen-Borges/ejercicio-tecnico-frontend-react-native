# AGENTS.md — Prueba Técnica Frontend (React Native)

Este archivo define la guía general para todos los agentes de IA que trabajan en este repositorio, siguiendo el flujo **ASDD (Agent Spec Software Development)**.

## Resumen del Proyecto

- **Framework**: React Native 0.73+
- **Lenguaje**: TypeScript 4.8+
- **React**: 18+
- **Navegación**: React Navigation v6 (Stack Navigator)
- **HTTP Client**: Fetch API nativo o Axios
- **Testing**: Jest + React Native Testing Library (coverage ≥ 70%)
- **Estilos**: StyleSheet de React Native — SIN frameworks externos (Tailwind, NativeBase, etc.)
- **Backend local**: Node.js en `http://localhost:3002`
- **Arquitectura**: services → hooks/state → components → screens/views

## Contexto de la Prueba Técnica

Este proyecto es una prueba técnica **Senior** que requiere:
- **F1**: Listado de productos financieros (GET /bp/products)
- **F2**: Búsqueda de productos por texto
- **F3**: Contador de registros
- **F4**: Agregar producto (POST /bp/products + validaciones + verificar ID)
- **F5**: Editar producto (PUT /bp/products/:id + formulario con ID deshabilitado)
- **F6**: Eliminar producto (DELETE /bp/products/:id + modal de confirmación)

## Flujo ASDD

**Todo feature nuevo sigue este pipeline:**

```
[FASE 1 — Secuencial]
spec-generator    → /generate-spec      → .github/specs/<feature>.spec.md

[FASE 2 — Paralelo ∥]
frontend-developer → screens / components / hooks / services (React Native + TS)

[FASE 3 — Paralelo ∥]
test-engineer-frontend → src/__tests__/ (Jest + RNTL, coverage ≥ 70%)

[FASE 4 — Secuencial]
qa-agent          → Gherkin, riesgos, colección Postman
```

> **No hay backend ni base de datos que implementar** — el backend corre localmente en http://localhost:3002.

## Reglas Críticas para Todos los Agentes

1. **Stack exclusivo**: React Native + TypeScript. NUNCA usar componentes de terceros (NativeBase, Tamagui, etc.) para estilos/UI.
2. **Estilos**: SIEMPRE usar `StyleSheet.create()`. Sin `style` inline con objetos crudos para lógica compleja.
3. **Sin spec APPROVED → sin implementación** — verificar `.github/specs/` primero.
4. **Navegación**: Usar React Navigation v6 con Stack Navigator. Props de navegación tipadas.
5. **TypeScript estricto**: Interfaces y tipos para todas las props, servicios y modelos. Sin `any`.
6. **Testing**: Jest + React Native Testing Library. Coverage mínimo **70%** (prueba técnica exige esto).
7. **SOLID + Clean Code**: Separación de responsabilidades entre capas services/hooks/components/screens.
8. **Manejo de errores**: Mostrar mensajes de error visuales al usuario — nunca silenciar errores.
9. **Skeletons**: Pantallas de precarga para mejorar UX (deseable en perfil Senior).

---
> Última actualización: 2026-05-10 — Stack migrado a React Native + TypeScript (Prueba Técnica Frontend 2024).
