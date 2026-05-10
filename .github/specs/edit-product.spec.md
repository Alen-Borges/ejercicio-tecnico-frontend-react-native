---
id: SPEC-003
status: APPROVED
feature: edit-product
created: 2026-05-10
updated: 2026-05-10
author: spec-generator
version: "1.0"
related-specs: [SPEC-001, SPEC-002]
---

# Spec: Editar Producto Financiero (F5)

> **Estado:** `APPROVED` — listo para implementar.

---

## 1. REQUERIMIENTOS

### Descripción

Formulario de edición de un producto financiero existente. Reutiliza `ProductForm` (creado en F4) con la diferencia clave de que el campo ID está **deshabilitado** — el ID no puede modificarse. Los datos del producto se pre-cargan en el formulario. Comparte las mismas validaciones de F4 (excepto la verificación de ID único).

### Historias de Usuario

#### HU-05: Editar producto financiero existente

```
Como:        Usuario de la app bancaria
Quiero:      Editar los datos de un producto financiero existente
Para:        Mantener actualizada la información del catálogo
Prioridad:   Alta | Estimación: M | Dependencias: HU-01, HU-04
```

**CRITERIO-5.1 (Happy Path)**
```gherkin
Dado que:  el usuario está en ProductDetailScreen de un producto
Cuando:    presiona el botón "Editar"
Entonces:  navega a EditProductScreen
Y:         el formulario tiene todos los campos pre-cargados con los datos actuales
Y:         el campo ID está deshabilitado (no editable)
```

**CRITERIO-5.2 (Happy Path — Guardar cambios)**
```gherkin
Dado que:  el usuario ha modificado campos válidos en EditProductScreen
Cuando:    presiona el botón "Actualizar"
Entonces:  se envía PUT /bp/products/:id con los datos modificados
Y:         el usuario regresa a ProductListScreen
Y:         los datos actualizados se reflejan en la lista
```

**CRITERIO-5.3 (Error Path — Validación)**
```gherkin
Dado que:  el usuario borra el contenido de un campo obligatorio
Cuando:    presiona "Actualizar"
Entonces:  se muestran mensajes de error rojos bajo los campos inválidos
Y:         NO se realiza ninguna llamada PUT
```

**CRITERIO-5.4 (Error Path — Error de API)**
```gherkin
Dado que:  el formulario tiene datos válidos
Cuando:    el PUT /bp/products/:id retorna 404 (producto no encontrado)
Entonces:  se muestra un mensaje de error visual al usuario
Y:         el usuario permanece en el formulario
```

### Reglas de Negocio

1. El campo **ID siempre está deshabilitado** — no se puede editar en modo edición.
2. Las validaciones son **idénticas a F4** excepto que NO se verifica unicidad del ID (el ID ya existe y no cambia).
3. `date_revision` sigue calculándose automáticamente al cambiar `date_release`.
4. El formulario se pre-carga con los valores actuales del producto al abrir la pantalla.
5. El endpoint PUT **no incluye el campo `id` en el body** — solo se envía en la URL.

---

## 2. DISEÑO

### API Endpoints Involucrados

| Método | Endpoint | Request Body | Response |
|--------|----------|-------------|---------|
| `PUT` | `/bp/products/:id` | `{ name, description, logo, date_release, date_revision }` | `200: { message, data }` / `404: { error }` |

**Request Body — PUT /bp/products/:id:**
```json
{
  "name": "string (5-100 chars)",
  "description": "string (10-200 chars)",
  "logo": "string (URL)",
  "date_release": "YYYY-MM-DD",
  "date_revision": "YYYY-MM-DD"
}
```

> Nota: El `id` NO va en el body, va en la URL.

### Screen: `EditProductScreen` (Ruta: `EditProduct`, param: `{ product: FinancialProduct }`)

Elementos UI (mismo diseño D2 que AddProductScreen):
- Header con título "Formulario de Edición" + botón back
- Campo **ID** (`TextInput`, `editable={false}`, estilo visual diferenciado — gris)
- Campo **Nombre** (`TextInput`) + mensaje de error
- Campo **Descripción** (`TextInput`, multiline) + mensaje de error
- Campo **Logo** (`TextInput`) + mensaje de error
- Campo **Fecha de Liberación** (`TextInput`) + mensaje de error
- Campo **Fecha de Revisión** (`TextInput`, readonly) + mensaje de error
- Botón **"Reiniciar"** — restaura todos los campos a los valores originales del producto
- Botón **"Actualizar"** — valida y envía el formulario

### Reutilización de `ProductForm`

`EditProductScreen` usa `ProductForm` con:
```typescript
<ProductForm
  initialValues={product}   // pre-carga datos del producto recibido por params
  onSubmit={handleUpdate}
  isEdit={true}             // desactiva el campo ID y omite la verificación de unicidad
  submitLabel="Actualizar"
  isLoading={loading}
/>
```

El botón "Reiniciar" en modo edición **restaura al `initialValues`** (no limpia a vacío).

### Service

```typescript
// src/services/productService.ts
export async function updateProduct(
  id: string,
  data: Omit<FinancialProduct, 'id'>
): Promise<FinancialProduct>;
```

### Hook extendido: `useProducts`

```typescript
updateProduct: (id: string, data: Omit<FinancialProduct, 'id'>) => Promise<void>;
// Internamente: llama productService.updateProduct → refresca lista
```

---

## 3. LISTA DE TAREAS

### Frontend Developer

- [ ] Crear `src/screens/EditProductScreen/EditProductScreen.tsx`
  - Recibir `route.params.product` (objeto `FinancialProduct`)
  - Usar `ProductForm` con `isEdit={true}`, `initialValues={product}`, `submitLabel="Actualizar"`
  - En `onSubmit`: valida, llama `updateProduct(product.id, data)`, navega a `ProductList`
  - Muestra loading durante el PUT
  - Muestra error de API si falla el PUT (404 o 400)
- [ ] Verificar que `ProductForm` (SPEC-002) soporte correctamente `isEdit={true}`:
  - Campo ID con `editable={false}` y estilo visual deshabilitado
  - Botón "Reiniciar" restaura a `initialValues` en lugar de vaciar
  - `validateProductForm` no valida ID en modo `isEdit=true`
- [ ] Extender `src/hooks/useProducts.ts` — agregar acción `updateProduct`
- [ ] Agregar `updateProduct` a `src/services/productService.ts` si no existe
- [ ] Verificar que la ruta `EditProduct` esté registrada en `AppNavigator.tsx` con param tipado

### Test Engineer Frontend

- [ ] `productService.test.ts` — updateProduct:
  - Happy path: envía PUT con URL correcta y body sin `id`, retorna `data`
  - Error 404: lanza error con mensaje apropiado
- [ ] `useProducts.test.ts` — updateProduct:
  - Actualiza el producto en la lista local tras éxito
  - Maneja error de updateProduct correctamente
- [ ] `ProductForm.test.tsx` (modo edición):
  - El campo ID está deshabilitado (`editable={false}`)
  - El campo ID muestra el valor inicial y no puede modificarse
  - Botón "Reiniciar" restaura a los valores iniciales (no a vacío)
  - Validaciones funcionan igual que en modo creación (excepto ID)
- [ ] `EditProductScreen.test.tsx`:
  - Renderiza el formulario con datos pre-cargados del producto
  - El campo ID está deshabilitado
  - Submit exitoso navega a ProductList
  - Muestra error si updateProduct falla

### QA

- [ ] `/gherkin-case-generator` → CRITERIO-5.1 a 5.4
- [ ] `/risk-identifier`
- [ ] `/postman-collection-generator` → PUT /bp/products/:id (con ID existente y no existente)
- [ ] Verificar cobertura ≥ 70%
- [ ] Actualizar estado: `status: IMPLEMENTED`
