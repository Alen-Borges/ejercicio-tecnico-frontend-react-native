---
applyTo: "src/**/*.{ts,tsx}"
---

> **Scope**: Se aplica a todos los archivos TypeScript/TSX del proyecto React Native. Mantener arquitectura limpia y separación de responsabilidades.

# Instrucciones para Archivos de Frontend (React Native + TypeScript)

## Contexto del Proyecto

Prueba técnica de una app para gestionar **Productos Financieros** de un banco.
- Backend local: `http://localhost:3002`
- Base URL de API: `http://localhost:3002/bp/products`
- Sin autenticación (la API es local, no requiere tokens)

## Convenciones Obligatorias

- **Componentes**: Componentes funcionales con Hooks. SIEMPRE tipados con `React.FC<Props>` o con tipado explícito de props.
- **Estilos**: SIEMPRE `StyleSheet.create()`. NUNCA frameworks externos (NativeBase, Tamagui, Paper, etc.).
- **Nombres**: PascalCase para componentes y screens (`.tsx`), camelCase para hooks (`useXxx.ts`) y servicios (`xxxService.ts`).
- **TypeScript**: Sin `any`. Usar interfaces y types explícitos para props, modelos y respuestas de API.
- **Variables de entorno**: La URL base de la API puede ir en un archivo de configuración central (`src/config/api.ts`).

## Modelo de Dominio

```typescript
// src/models/FinancialProduct.ts
export interface FinancialProduct {
  id: string;          // Identificador único (3–10 chars)
  name: string;        // Nombre (5–100 chars)
  description: string; // Descripción (10–200 chars)
  logo: string;        // URL de imagen representativa
  date_release: string; // Fecha ISO (>= hoy)
  date_revision: string; // Fecha ISO (exactamente 1 año después de date_release)
}
```

## Estructura de Archivos del Proyecto

```
src/
  config/
    api.ts              ← URL base y configuración HTTP
  models/
    FinancialProduct.ts ← Interfaces/types del dominio
  services/
    productService.ts   ← Llamadas HTTP (GET, POST, PUT, DELETE, verificar ID)
  hooks/
    useProducts.ts      ← Estado, loading, error y acciones CRUD
  components/
    ProductList/        ← Lista de productos con búsqueda y contador
    ProductCard/        ← Tarjeta individual de producto
    ProductForm/        ← Formulario reutilizable (agregar/editar)
    DeleteModal/        ← Modal de confirmación de eliminación
    SkeletonLoader/     ← Pantallas de precarga (deseable Senior)
    ErrorMessage/       ← Componente visual de error
  screens/
    ProductListScreen/  ← F1 + F2 + F3: Listado, búsqueda, contador
    ProductDetailScreen/ ← Detalle del producto seleccionado
    AddProductScreen/   ← F4: Formulario de agregar
    EditProductScreen/  ← F5: Formulario de editar (ID deshabilitado)
  navigation/
    AppNavigator.tsx    ← Stack Navigator con todas las rutas tipadas
  App.tsx              ← Entry point
```

## API del Backend Local

```typescript
// src/services/productService.ts
const BASE_URL = 'http://localhost:3002';

// GET  /bp/products             → { data: FinancialProduct[] }
// POST /bp/products             → { message, data: FinancialProduct }
// PUT  /bp/products/:id         → { message, data: FinancialProduct }
// DELETE /bp/products/:id       → { message: string }
// GET  /bp/products/verification/:id → true | false
```

## Llamadas HTTP — Patrón de Servicio

```typescript
// services/productService.ts
export async function getProducts(): Promise<FinancialProduct[]> {
  const res = await fetch(`${BASE_URL}/bp/products`);
  if (!res.ok) throw new Error(`Error ${res.status}`);
  const json = await res.json();
  return json.data;
}

export async function verifyProductId(id: string): Promise<boolean> {
  const res = await fetch(`${BASE_URL}/bp/products/verification/${id}`);
  if (!res.ok) throw new Error(`Error ${res.status}`);
  return res.json(); // true o false
}
```

## Navegación (React Navigation v6)

```typescript
// navigation/AppNavigator.tsx
export type RootStackParamList = {
  ProductList: undefined;
  ProductDetail: { product: FinancialProduct };
  AddProduct: undefined;
  EditProduct: { product: FinancialProduct };
};
```

## Validaciones del Formulario (F4 / F5)

| Campo | Regla |
|-------|-------|
| `id` | Requerido, 3–10 chars, no debe existir (verificar API) |
| `name` | Requerido, 5–100 chars |
| `description` | Requerido, 10–200 chars |
| `logo` | Requerido |
| `date_release` | Requerido, >= fecha actual |
| `date_revision` | Requerido, exactamente +1 año de `date_release` |

## Manejo de Errores

- **SIEMPRE** mostrar mensajes de error visuales al usuario (componente `ErrorMessage` o texto rojo bajo el campo).
- Errores de validación: mostrar bajo cada campo del formulario.
- Errores de red/API: mostrar toast o alerta inline en la pantalla.

## Nunca hacer

- Importar librerías de UI externas (NativeBase, Paper, Tamagui, etc.).
- Usar `any` en TypeScript.
- Llamadas HTTP directamente desde componentes — siempre usar la capa de servicios.
- Lógica de negocio en la capa de presentación (screens/components).
- Swallow errors (ignorar errores sin feedback al usuario).

---

> Para estándares de testing, ver `.github/instructions/tests.instructions.md`.
> Para lineamientos de calidad, ver `.github/docs/lineamientos/dev-guidelines.md`.
