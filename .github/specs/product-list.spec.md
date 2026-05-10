---
id: SPEC-001
status: APPROVED
feature: product-list
created: 2026-05-10
updated: 2026-05-10
author: spec-generator
version: "1.0"
related-specs: [SPEC-002, SPEC-003, SPEC-004]
---

# Spec: Listado, Búsqueda y Contador de Productos Financieros (F1 + F2 + F3)

> **Estado:** `APPROVED` — listo para implementar.

---

## 1. REQUERIMIENTOS

### Descripción

Pantalla principal que lista todos los productos financieros desde la API local. Permite buscar por texto en tiempo real y muestra el contador de registros. Al tocar un producto navega al detalle.

### Historias de Usuario

#### HU-01: Listar productos financieros
```
Como:        Usuario de la app bancaria
Quiero:      Ver la lista de todos los productos financieros
Para:        Conocer la oferta de productos del banco
Prioridad:   Alta | Estimación: M | Capa: Frontend
```

**CRITERIO-1.1 (Happy Path)**
```gherkin
Dado que:  el backend local está corriendo en http://localhost:3002
Cuando:    el usuario abre ProductListScreen
Entonces:  se muestra la lista de productos (logo, nombre, ID)
Y:         se muestra el contador con el total de registros
```

**CRITERIO-1.2 (Error Path)**
```gherkin
Dado que:  el backend no está disponible
Cuando:    la app intenta cargar los productos
Entonces:  se muestra un mensaje de error visual al usuario
```

**CRITERIO-1.3 (Edge Case)**
```gherkin
Dado que:  el backend responde con array data vacío
Cuando:    la app renderiza la pantalla
Entonces:  se muestra mensaje "no hay productos" y contador "0 resultados"
```

#### HU-02: Buscar productos por texto
```
Como:        Usuario de la app bancaria
Quiero:      Filtrar productos mediante un campo de texto
Para:        Encontrar rápidamente el producto que busco
Prioridad:   Alta | Estimación: S | Dependencias: HU-01
```

**CRITERIO-2.1 (Happy Path)**
```gherkin
Dado que:  hay N productos cargados
Cuando:    el usuario escribe "tarjeta" en el campo de búsqueda
Entonces:  solo se muestran productos que contienen "tarjeta" en nombre o descripción
Y:         el contador se actualiza con la cantidad filtrada
```

**CRITERIO-2.2 (Edge Case)**
```gherkin
Dado que:  hay N productos cargados
Cuando:    el usuario escribe texto sin coincidencias
Entonces:  el listado queda vacío y el contador muestra "0 resultados"
```

#### HU-03: Contador de registros
```
Como:        Usuario de la app bancaria
Quiero:      Ver cuántos productos se están mostrando
Para:        Saber el tamaño del resultado (con o sin filtro)
Prioridad:   Alta | Estimación: XS | Dependencias: HU-01, HU-02
```

**CRITERIO-3.1**
```gherkin
Dado que:  hay 5 productos cargados y el filtro retorna 3
Cuando:    el usuario mira el contador
Entonces:  el contador muestra "3 resultados"
```

### Reglas de Negocio

1. El filtrado es **local** — no hace nueva llamada a la API por keystroke.
2. La búsqueda es **case-insensitive** y compara `name` + `description`.
3. El contador muestra siempre los ítems **visibles** post-filtro.
4. Errores de red se muestran visualmente — nunca silenciados.
5. Mientras carga: mostrar SkeletonLoader o ActivityIndicator.
6. Al tocar un ítem → navegar a `ProductDetailScreen` con el objeto `product`.

---

## 2. DISEÑO

### Modelo de Dominio

```typescript
// src/models/FinancialProduct.ts
export interface FinancialProduct {
  id: string;
  name: string;
  description: string;
  logo: string;
  date_release: string;
  date_revision: string;
}
```

### API

| Método | Endpoint | Response 200 |
|--------|----------|-------------|
| `GET` | `/bp/products` | `{ data: FinancialProduct[] }` |

### Screens

#### `ProductListScreen` (Ruta: `ProductList`)
- Header con título "BANCO"
- `TextInput` de búsqueda
- Texto contador: `N resultados`
- `FlatList` de `ProductCard` (keyExtractor: `product.id`)
- Botón "Agregar" → navega a `AddProduct`
- Estados: loading (Skeleton), error (ErrorMessage), vacío (EmptyState)

#### `ProductDetailScreen` (Ruta: `ProductDetail`, param: `{ product }`)
- Logo, ID, Nombre, Descripción, Fecha liberación, Fecha revisión
- Botón "Editar" → `EditProduct`
- Botón "Eliminar" → mostrar `DeleteModal`

### Componentes

| Componente | Archivo | Props |
|------------|---------|-------|
| `ProductCard` | `src/components/ProductCard/ProductCard.tsx` | `product, onPress, testID?` |
| `SkeletonLoader` | `src/components/SkeletonLoader/SkeletonLoader.tsx` | `count?: number` |
| `ErrorMessage` | `src/components/ErrorMessage/ErrorMessage.tsx` | `message, onRetry?` |
| `EmptyState` | `src/components/EmptyState/EmptyState.tsx` | `message` |

### Hook

```typescript
// src/hooks/useProducts.ts
// Retorna:
{
  products: FinancialProduct[];
  filteredProducts: FinancialProduct[]; // useMemo derivado
  loading: boolean;
  error: string | null;
  searchQuery: string;
  setSearchQuery: (q: string) => void;
  loadProducts: () => Promise<void>;
  createProduct: (data: Omit<FinancialProduct, 'id'> & { id: string }) => Promise<void>;
  updateProduct: (id: string, data: Partial<FinancialProduct>) => Promise<void>;
  deleteProduct: (id: string) => Promise<void>;
}
```

### Service

```typescript
// src/services/productService.ts
export async function getProducts(): Promise<FinancialProduct[]>;
export async function createProduct(data: FinancialProduct): Promise<FinancialProduct>;
export async function updateProduct(id: string, data: Partial<FinancialProduct>): Promise<FinancialProduct>;
export async function deleteProduct(id: string): Promise<void>;
export async function verifyProductId(id: string): Promise<boolean>;
```

### Navegación

```typescript
// src/navigation/AppNavigator.tsx
export type RootStackParamList = {
  ProductList: undefined;
  ProductDetail: { product: FinancialProduct };
  AddProduct: undefined;
  EditProduct: { product: FinancialProduct };
};
```

---

## 3. LISTA DE TAREAS

### Frontend Developer

- [ ] Crear `src/config/api.ts` — `BASE_URL = 'http://localhost:3002'`
- [ ] Crear `src/models/FinancialProduct.ts` — interfaz tipada
- [ ] Crear `src/services/productService.ts` — todas las funciones (getProducts, createProduct, updateProduct, deleteProduct, verifyProductId)
- [ ] Crear `src/hooks/useProducts.ts` — estado + filteredProducts (useMemo) + acciones CRUD
- [ ] Crear `src/components/ProductCard/ProductCard.tsx` — logo, nombre, ID, fecha, testID
- [ ] Crear `src/components/SkeletonLoader/SkeletonLoader.tsx` — placeholder animado
- [ ] Crear `src/components/ErrorMessage/ErrorMessage.tsx` — texto rojo + reintentar
- [ ] Crear `src/components/EmptyState/EmptyState.tsx` — texto vacío
- [ ] Crear `src/screens/ProductListScreen/ProductListScreen.tsx` — FlatList + búsqueda + contador + botón Agregar
- [ ] Crear `src/screens/ProductDetailScreen/ProductDetailScreen.tsx` — todos los campos + botones
- [ ] Crear `src/navigation/AppNavigator.tsx` — NativeStackNavigator con RootStackParamList
- [ ] Actualizar `src/App.tsx` — wrappear con NavigationContainer + AppNavigator

### Test Engineer Frontend

- [ ] `productService.test.ts` — getProducts: happy path, error HTTP, error de red
- [ ] `useProducts.test.ts` — loading inicial, carga exitosa, error, filtro por nombre, filtro por descripción, sin resultados
- [ ] `ProductCard.test.tsx` — renderiza nombre, llama onPress
- [ ] `ErrorMessage.test.tsx` — renderiza mensaje, botón reintentar
- [ ] `ProductListScreen.test.tsx` — lista productos, contador, skeleton en loading, error state, navegación al detalle
- [ ] `ProductDetailScreen.test.tsx` — renderiza todos los campos, botón editar, botón eliminar

### QA

- [ ] `/gherkin-case-generator` → CRITERIO-1.1 a 3.1
- [ ] `/risk-identifier` → matriz de riesgos
- [ ] `/postman-collection-generator` → GET /bp/products
- [ ] Verificar cobertura ≥ 70%
- [ ] Actualizar estado: `status: IMPLEMENTED`
