---
id: SPEC-###
status: DRAFT
feature: nombre-del-feature
created: YYYY-MM-DD
updated: YYYY-MM-DD
author: spec-generator
version: "1.0"
related-specs: []
---

# Spec: [Nombre de la Funcionalidad]

> **Estado:** `DRAFT` → aprobar con `status: APPROVED` antes de iniciar implementación.
> **Ciclo de vida:** DRAFT → APPROVED → IN_PROGRESS → IMPLEMENTED → DEPRECATED

---

## 1. REQUERIMIENTOS

### Descripción
Resumen de la funcionalidad en 2-3 oraciones. Qué hace, para quién y qué problema resuelve.

### Requerimiento de Negocio
El requerimiento original tal como fue proporcionado (copiado de `.github/requirements/<feature>.md` o de la prueba técnica).

### Historias de Usuario

#### HU-01: [Título descriptivo corto]

```
Como:        [rol del usuario — ej. Usuario de la app bancaria]
Quiero:      [acción o funcionalidad concreta]
Para:        [valor o beneficio esperado]

Prioridad:   Alta / Media / Baja
Estimación:  XS / S / M / L / XL
Dependencias: HU-X, HU-Y o Ninguna
Capa:        Frontend (React Native)
```

#### Criterios de Aceptación — HU-01

**Happy Path**
```gherkin
CRITERIO-1.1: [nombre del escenario exitoso]
  Dado que:  [contexto inicial válido]
  Cuando:    [acción del usuario en la app]
  Entonces:  [resultado esperado verificable en la UI]
```

**Error Path**
```gherkin
CRITERIO-1.2: [nombre del escenario de error]
  Dado que:  [contexto inicial]
  Cuando:    [acción inválida o datos incorrectos]
  Entonces:  [mensaje de error visible al usuario]
```

**Edge Case** *(si aplica)*
```gherkin
CRITERIO-1.3: [nombre del caso borde]
  Dado que:  [contexto de borde]
  Cuando:    [acción en el límite]
  Entonces:  [resultado esperado en el límite]
```

### Reglas de Negocio
1. Regla de validación (ej. "el campo ID es requerido, 3–10 chars, debe ser único")
2. Regla de UX (ej. "mostrar error visual inmediato bajo el campo inválido")
3. Regla de navegación (ej. "al eliminar, redirigir al listado")

---

## 2. DISEÑO

### Modelo de Dominio

```typescript
// src/models/FinancialProduct.ts
interface FinancialProduct {
  id: string;           // Identificador único del producto (3–10 chars)
  name: string;         // Nombre del Producto (5–100 chars)
  description: string;  // Descripción (10–200 chars)
  logo: string;         // URL de un logo representativo
  date_release: string; // Fecha ISO: igual o mayor a hoy
  date_revision: string; // Fecha ISO: exactamente +1 año de date_release
}
```

### API Endpoints Involucrados

| Método | Endpoint | Uso |
|--------|----------|-----|
| `GET` | `/bp/products` | Listar todos los productos |
| `POST` | `/bp/products` | Crear nuevo producto |
| `PUT` | `/bp/products/:id` | Actualizar producto existente |
| `DELETE` | `/bp/products/:id` | Eliminar producto |
| `GET` | `/bp/products/verification/:id` | Verificar si ID ya existe (true/false) |

#### Detalle del endpoint principal

**[MÉTODO] [endpoint]**
- **Request Body** (si aplica):
  ```json
  {
    "id": "string",
    "name": "string",
    "description": "string",
    "logo": "string",
    "date_release": "YYYY-MM-DD",
    "date_revision": "YYYY-MM-DD"
  }
  ```
- **Response 200**:
  ```json
  { "message": "...", "data": { /* FinancialProduct */ } }
  ```
- **Response 400**: body inválido o validación fallida
- **Response 404**: producto no encontrado

### Diseño de Screens

#### Screens afectadas / nuevas

| Screen | Archivo | Ruta Navigation | Descripción |
|--------|---------|----------------|-------------|
| `ProductListScreen` | `src/screens/ProductListScreen/` | `ProductList` | Listado + búsqueda + contador |
| `ProductDetailScreen` | `src/screens/ProductDetailScreen/` | `ProductDetail` | Detalle del producto seleccionado |
| `AddProductScreen` | `src/screens/AddProductScreen/` | `AddProduct` | Formulario de creación |
| `EditProductScreen` | `src/screens/EditProductScreen/` | `EditProduct` | Formulario de edición (ID readonly) |

#### Componentes nuevos / afectados

| Componente | Archivo | Props principales | Descripción |
|------------|---------|------------------|-------------|
| `ProductCard` | `src/components/ProductCard/` | `product, onPress` | Tarjeta de producto en la lista |
| `ProductForm` | `src/components/ProductForm/` | `initialValues, onSubmit, isEdit` | Formulario reutilizable (add/edit) |
| `DeleteModal` | `src/components/DeleteModal/` | `visible, productName, onConfirm, onCancel` | Modal de confirmación de eliminación |
| `SkeletonLoader` | `src/components/SkeletonLoader/` | `count` | Pantalla de carga (deseable Senior) |

#### Hooks / State

| Hook | Archivo | Retorna | Descripción |
|------|---------|---------|-------------|
| `useProducts` | `src/hooks/useProducts.ts` | `{ products, loading, error, loadProducts, createProduct, updateProduct, deleteProduct, searchQuery, setSearchQuery, filteredProducts }` | CRUD + filtro |

#### Services (llamadas API)

| Función | Archivo | Endpoint |
|---------|---------|---------|
| `getProducts()` | `src/services/productService.ts` | `GET /bp/products` |
| `createProduct(data)` | `src/services/productService.ts` | `POST /bp/products` |
| `updateProduct(id, data)` | `src/services/productService.ts` | `PUT /bp/products/:id` |
| `deleteProduct(id)` | `src/services/productService.ts` | `DELETE /bp/products/:id` |
| `verifyProductId(id)` | `src/services/productService.ts` | `GET /bp/products/verification/:id` |

### Navegación

```typescript
// Parámetros de ruta
type RootStackParamList = {
  ProductList: undefined;
  ProductDetail: { product: FinancialProduct };
  AddProduct: undefined;
  EditProduct: { product: FinancialProduct };
};
```

### Notas de Implementación
> Observaciones técnicas, decisiones de diseño o advertencias para el Frontend Developer.
> Ej. "El campo `date_revision` debe calcularse automáticamente al cambiar `date_release`."

---

## 3. LISTA DE TAREAS

> Checklist accionable para todos los agentes. Marcar cada ítem (`[x]`) al completarlo.
> El Orchestrator monitorea este checklist para determinar el progreso.

### Frontend Developer

#### Services
- [ ] Implementar `getProducts()` en `src/services/productService.ts`
- [ ] Implementar `createProduct(data)` — POST con body tipado
- [ ] Implementar `updateProduct(id, data)` — PUT con ID en URL
- [ ] Implementar `deleteProduct(id)` — DELETE con ID en URL
- [ ] Implementar `verifyProductId(id)` — retorna `true` / `false`

#### Hooks / State
- [ ] Implementar `useProducts` con: `products`, `loading`, `error`, `loadProducts`
- [ ] Agregar `searchQuery` + `filteredProducts` al hook (F2)
- [ ] Agregar acciones: `createProduct`, `updateProduct`, `deleteProduct`

#### Components
- [ ] Implementar `ProductCard` — render con logo, nombre, fecha liberación, onPress
- [ ] Implementar `ProductForm` — todos los campos con validaciones + botones Agregar/Reiniciar
- [ ] Implementar `DeleteModal` — texto con nombre del producto, botones Confirmar/Cancelar
- [ ] Implementar `SkeletonLoader` — N placeholders animados (deseable Senior)
- [ ] Implementar `ErrorMessage` — componente de error visual reutilizable

#### Screens
- [ ] Implementar `ProductListScreen` — lista, búsqueda TextInput, contador (F1+F2+F3)
- [ ] Implementar `ProductDetailScreen` — datos completos del producto seleccionado
- [ ] Implementar `AddProductScreen` — formulario de creación (F4)
- [ ] Implementar `EditProductScreen` — formulario de edición con ID readonly (F5)

#### Navegación
- [ ] Registrar todas las rutas en `src/navigation/AppNavigator.tsx`
- [ ] Tipar `RootStackParamList` con todos los params necesarios

### Test Engineer Frontend

#### Services
- [ ] `getProducts` happy path retorna array
- [ ] `getProducts` error HTTP lanza excepción con código
- [ ] `createProduct` envía body correcto
- [ ] `deleteProduct` llama endpoint correcto
- [ ] `verifyProductId` retorna `true` cuando existe
- [ ] `verifyProductId` retorna `false` cuando no existe

#### Hooks
- [ ] `useProducts` carga productos en mount
- [ ] `useProducts` maneja error de red correctamente
- [ ] `useProducts` filtra por `searchQuery`
- [ ] `useProducts` actualiza lista tras crear/editar/eliminar

#### Components
- [ ] `ProductCard` renderiza nombre del producto
- [ ] `ProductCard` llama `onPress` al presionar
- [ ] `ProductForm` muestra error cuando ID < 3 chars
- [ ] `ProductForm` muestra error cuando nombre < 5 chars
- [ ] `ProductForm` muestra error cuando fecha de liberación es en el pasado
- [ ] `ProductForm` calcula automáticamente fecha de revisión
- [ ] `ProductForm` botón Reiniciar limpia todos los campos
- [ ] `DeleteModal` llama `onConfirm` al presionar Confirmar
- [ ] `DeleteModal` llama `onCancel` al presionar Cancelar

#### Screens
- [ ] `ProductListScreen` renderiza lista de productos
- [ ] `ProductListScreen` muestra contador correcto
- [ ] `AddProductScreen` valida todos los campos antes de enviar
- [ ] `EditProductScreen` tiene campo ID deshabilitado
- [ ] `EditProductScreen` pre-carga datos del producto a editar

### QA

- [ ] Ejecutar `/gherkin-case-generator` → criterios CRITERIO-1.1, 1.2, 1.3
- [ ] Ejecutar `/risk-identifier` → clasificación de riesgos
- [ ] Ejecutar `/postman-collection-generator` → colección para backend local
- [ ] Verificar cobertura ≥ 70% con `npx jest --coverage`
- [ ] Actualizar estado spec: `status: IMPLEMENTED`
