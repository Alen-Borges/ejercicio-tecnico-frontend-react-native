---
id: SPEC-002
status: APPROVED
feature: add-product
created: 2026-05-10
updated: 2026-05-10
author: spec-generator
version: "1.0"
related-specs: [SPEC-001, SPEC-003]
---

# Spec: Agregar Producto Financiero (F4)

> **Estado:** `APPROVED` — listo para implementar.

---

## 1. REQUERIMIENTOS

### Descripción

Formulario para crear un nuevo producto financiero. Se accede desde el botón "Agregar" del listado. Incluye validaciones campo a campo con mensajes de error visuales, verificación de ID único vía API, botón "Agregar" para enviar y botón "Reiniciar" para limpiar el formulario.

### Historias de Usuario

#### HU-04: Agregar producto financiero con datos válidos

```
Como:        Usuario de la app bancaria
Quiero:      Crear un nuevo producto financiero mediante un formulario
Para:        Incorporar un nuevo producto al catálogo del banco
Prioridad:   Alta | Estimación: L | Dependencias: HU-01
```

**CRITERIO-4.1 (Happy Path)**
```gherkin
Dado que:  el usuario navega a AddProductScreen
Cuando:    llena todos los campos con datos válidos y presiona "Agregar"
Entonces:  se envía POST /bp/products con los datos del formulario
Y:         el usuario regresa a ProductListScreen
Y:         el nuevo producto aparece en la lista
```

**CRITERIO-4.2 (Error Path — Validación de campos)**
```gherkin
Dado que:  el usuario presiona "Agregar" sin llenar los campos obligatorios
Cuando:    se ejecuta la validación del formulario
Entonces:  se muestran mensajes de error rojos bajo cada campo inválido
Y:         NO se realiza ninguna llamada a la API
```

**CRITERIO-4.3 (Error Path — ID duplicado)**
```gherkin
Dado que:  el usuario escribe un ID que ya existe en el sistema
Cuando:    el campo ID pierde el foco o se presiona "Agregar"
Entonces:  se muestra error visual: "Este ID ya existe"
Y:         NO se realiza el POST
```

**CRITERIO-4.4 (Error Path — Error de API)**
```gherkin
Dado que:  el formulario tiene datos válidos
Cuando:    el POST /bp/products retorna error 400
Entonces:  se muestra un mensaje de error visual al usuario
Y:         el usuario permanece en el formulario para corregir
```

**CRITERIO-4.5 (Reiniciar formulario)**
```gherkin
Dado que:  el usuario ha llenado varios campos del formulario
Cuando:    presiona el botón "Reiniciar"
Entonces:  todos los campos vuelven a su estado vacío/inicial
Y:         los mensajes de error desaparecen
```

### Reglas de Negocio — Validaciones por campo

| Campo | Regla | Mensaje de Error |
|-------|-------|-----------------|
| **ID** | Requerido | "El ID es requerido" |
| **ID** | Mínimo 3 caracteres | "Mínimo 3 caracteres" |
| **ID** | Máximo 10 caracteres | "Máximo 10 caracteres" |
| **ID** | No debe existir (API check) | "Este ID ya existe" |
| **Nombre** | Requerido | "El nombre es requerido" |
| **Nombre** | Mínimo 5 caracteres | "Mínimo 5 caracteres" |
| **Nombre** | Máximo 100 caracteres | "Máximo 100 caracteres" |
| **Descripción** | Requerido | "La descripción es requerida" |
| **Descripción** | Mínimo 10 caracteres | "Mínimo 10 caracteres" |
| **Descripción** | Máximo 200 caracteres | "Máximo 200 caracteres" |
| **Logo** | Requerido | "El logo es requerido" |
| **Fecha de Liberación** | Requerido | "La fecha de liberación es requerida" |
| **Fecha de Liberación** | >= fecha actual | "La fecha debe ser igual o mayor a hoy" |
| **Fecha de Revisión** | Requerido | "La fecha de revisión es requerida" |
| **Fecha de Revisión** | Exactamente +1 año de date_release | "Debe ser exactamente un año posterior a la fecha de liberación" |

> **Nota**: La `date_revision` se calcula y completa automáticamente cuando el usuario ingresa `date_release`.

---

## 2. DISEÑO

### API Endpoints Involucrados

| Método | Endpoint | Uso | Response |
|--------|----------|-----|---------|
| `POST` | `/bp/products` | Crear producto | `200: { message, data }` / `400: { error }` |
| `GET` | `/bp/products/verification/:id` | Verificar si ID existe | `200: true / false` |

**Request Body — POST /bp/products:**
```json
{
  "id": "string (3-10 chars)",
  "name": "string (5-100 chars)",
  "description": "string (10-200 chars)",
  "logo": "string (URL)",
  "date_release": "YYYY-MM-DD",
  "date_revision": "YYYY-MM-DD"
}
```

### Screen: `AddProductScreen` (Ruta: `AddProduct`)

Elementos UI (Diseño D2 + botón en Diseño D3):
- Header con título "Formulario de Registro" + botón back
- Campo **ID** (`TextInput`) + mensaje de error
- Campo **Nombre** (`TextInput`) + mensaje de error
- Campo **Descripción** (`TextInput`, multiline) + mensaje de error
- Campo **Logo** (`TextInput`) + mensaje de error
- Campo **Fecha de Liberación** (`TextInput` o DatePicker) + mensaje de error
- Campo **Fecha de Revisión** (`TextInput`, readonly auto-calculado) + mensaje de error
- Botón **"Reiniciar"** — limpia todos los campos y errores
- Botón **"Agregar"** — valida y envía el formulario

### Componente Reutilizable: `ProductForm`

```typescript
// src/components/ProductForm/ProductForm.tsx
interface ProductFormProps {
  initialValues?: Partial<FinancialProduct>;
  onSubmit: (data: FinancialProduct) => Promise<void>;
  isEdit?: boolean;        // Si true: campo ID deshabilitado (F5)
  submitLabel?: string;    // "Agregar" | "Actualizar"
  isLoading?: boolean;
}
```

### Lógica de Validación (función pura, exportable y testeable)

```typescript
// src/utils/productValidation.ts
export interface ProductFormErrors {
  id?: string;
  name?: string;
  description?: string;
  logo?: string;
  date_release?: string;
  date_revision?: string;
}

export function validateProductForm(
  data: Partial<FinancialProduct>,
  isEdit = false
): ProductFormErrors {
  const errors: ProductFormErrors = {};
  const today = new Date().toISOString().split('T')[0];

  if (!isEdit) {
    if (!data.id) errors.id = 'El ID es requerido';
    else if (data.id.length < 3) errors.id = 'Mínimo 3 caracteres';
    else if (data.id.length > 10) errors.id = 'Máximo 10 caracteres';
  }

  if (!data.name) errors.name = 'El nombre es requerido';
  else if (data.name.length < 5) errors.name = 'Mínimo 5 caracteres';
  else if (data.name.length > 100) errors.name = 'Máximo 100 caracteres';

  if (!data.description) errors.description = 'La descripción es requerida';
  else if (data.description.length < 10) errors.description = 'Mínimo 10 caracteres';
  else if (data.description.length > 200) errors.description = 'Máximo 200 caracteres';

  if (!data.logo) errors.logo = 'El logo es requerido';

  if (!data.date_release) errors.date_release = 'La fecha de liberación es requerida';
  else if (data.date_release < today) errors.date_release = 'La fecha debe ser igual o mayor a hoy';

  if (!data.date_revision) errors.date_revision = 'La fecha de revisión es requerida';
  else if (data.date_release) {
    const release = new Date(data.date_release);
    release.setFullYear(release.getFullYear() + 1);
    const expected = release.toISOString().split('T')[0];
    if (data.date_revision !== expected)
      errors.date_revision = 'Debe ser exactamente un año posterior a la fecha de liberación';
  }

  return errors;
}

export function calcRevisionDate(releaseDate: string): string {
  if (!releaseDate) return '';
  const d = new Date(releaseDate);
  d.setFullYear(d.getFullYear() + 1);
  return d.toISOString().split('T')[0];
}
```

### Hook extendido: `useProducts` (acciones CRUD)

```typescript
// Agregar a useProducts.ts:
createProduct: (data: FinancialProduct) => Promise<void>;
// Internamente: llama productService.createProduct → refresca lista → navega atrás
```

---

## 3. LISTA DE TAREAS

### Frontend Developer

- [ ] Crear `src/utils/productValidation.ts` — `validateProductForm` + `calcRevisionDate` (funciones puras)
- [ ] Crear `src/components/ProductForm/ProductForm.tsx` — formulario reutilizable con todos los campos, validaciones visuales, props: `initialValues`, `onSubmit`, `isEdit`, `submitLabel`, `isLoading`
  - Cada campo tiene `TextInput` + texto de error rojo debajo
  - Campo `date_revision` es readonly y se auto-calcula al cambiar `date_release`
  - testIDs: `input-id`, `input-name`, `input-description`, `input-logo`, `input-date-release`, `input-date-revision`, `btn-submit`, `btn-reset`
- [ ] Crear `src/screens/AddProductScreen/AddProductScreen.tsx`
  - Usa `ProductForm` con `isEdit={false}`, `submitLabel="Agregar"`
  - En `onSubmit`: valida, verifica ID vía `verifyProductId`, llama `createProduct`, navega a `ProductList`
  - Muestra estado de loading en el botón mientras envía
  - Muestra error de API visualmente si falla el POST
- [ ] Extender `src/hooks/useProducts.ts` — agregar acción `createProduct`
- [ ] Agregar `createProduct` a `src/services/productService.ts` si no existe

### Test Engineer Frontend

- [ ] `productValidation.test.ts` — validateProductForm:
  - ID vacío → error requerido
  - ID de 2 chars → error mínimo 3
  - ID de 11 chars → error máximo 10
  - Nombre vacío → error requerido
  - Nombre de 4 chars → error mínimo 5
  - Descripción de 9 chars → error mínimo 10
  - Logo vacío → error requerido
  - date_release pasada → error fecha
  - date_revision no es +1 año → error
  - Todos los campos válidos → sin errores `{}`
- [ ] `productValidation.test.ts` — calcRevisionDate:
  - `calcRevisionDate('2025-06-15')` → `'2026-06-15'`
  - `calcRevisionDate('')` → `''`
- [ ] `ProductForm.test.tsx`:
  - Renderiza todos los campos
  - Botón "Reiniciar" limpia todos los campos
  - Muestra errores de validación al presionar submit sin datos
  - date_revision se auto-calcula al cambiar date_release
- [ ] `productService.test.ts` — createProduct:
  - Happy path: envía POST con body correcto, retorna `data`
  - Error 400: lanza error
- [ ] `productService.test.ts` — verifyProductId:
  - Retorna `true` cuando ID existe
  - Retorna `false` cuando ID no existe
- [ ] `AddProductScreen.test.tsx`:
  - Renderiza el formulario
  - Muestra error si ID ya existe (mock verifyProductId → true)
  - Submit exitoso navega a ProductList
  - Muestra error de API si createProduct falla

### QA

- [ ] `/gherkin-case-generator` → CRITERIO-4.1 a 4.5
- [ ] `/risk-identifier`
- [ ] `/postman-collection-generator` → POST /bp/products + GET /bp/products/verification/:id
- [ ] Verificar cobertura ≥ 70%
- [ ] Actualizar estado: `status: IMPLEMENTED`
