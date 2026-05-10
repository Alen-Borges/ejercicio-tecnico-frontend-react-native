---
id: SPEC-004
status: APPROVED
feature: delete-product
created: 2026-05-10
updated: 2026-05-10
author: spec-generator
version: "1.0"
related-specs: [SPEC-001]
---

# Spec: Eliminar Producto Financiero (F6)

> **Estado:** `APPROVED` — listo para implementar.

---

## 1. REQUERIMIENTOS

### Descripción

Funcionalidad para eliminar un producto financiero del sistema. Al presionar el botón "Eliminar" en la pantalla de detalle, se debe mostrar un modal de confirmación con el nombre del producto. Si se confirma, el producto se elimina mediante la API y el usuario regresa al listado.

### Historias de Usuario

#### HU-06: Eliminar producto financiero

```
Como:        Usuario de la app bancaria
Quiero:      Eliminar un producto financiero que ya no es necesario
Para:        Mantener actualizado y limpio el catálogo del banco
Prioridad:   Alta | Estimación: S | Dependencias: HU-01
```

**CRITERIO-6.1 (Happy Path — Modal de confirmación)**
```gherkin
Dado que:  el usuario está en ProductDetailScreen de un producto
Cuando:    presiona el botón "Eliminar"
Entonces:  se muestra un modal de confirmación (Diseño D4)
Y:         el modal muestra el texto: "¿Estás seguro de eliminar el producto {nombre}?"
Y:         el modal tiene botones "Eliminar" (Confirmar) y "Cancelar"
```

**CRITERIO-6.2 (Happy Path — Confirmar eliminación)**
```gherkin
Dado que:  el modal de confirmación está visible
Cuando:    el usuario presiona el botón "Eliminar" (Confirmar)
Entonces:  se cierra el modal
Y:         se envía DELETE /bp/products/:id
Y:         se muestra un mensaje de éxito o se redirige a ProductListScreen
Y:         el producto ya no aparece en la lista
```

**CRITERIO-6.3 (Happy Path — Cancelar eliminación)**
```gherkin
Dado que:  el modal de confirmación está visible
Cuando:    el usuario presiona el botón "Cancelar"
Entonces:  el modal se cierra
Y:         el producto permanece sin cambios
```

**CRITERIO-6.4 (Error Path — Error de API)**
```gherkin
Dado que:  el usuario confirma la eliminación
Cuando:    el DELETE /bp/products/:id retorna un error (ej. 404)
Entonces:  se muestra un mensaje de error visual al usuario
Y:         el usuario permanece en la pantalla de detalle
```

### Reglas de Negocio

1. El modal debe ser centrado y bloquear la interacción con el fondo.
2. El botón "Confirmar" debe ejecutar la llamada a la API y el botón "Cancelar" simplemente ocultar el modal.
3. Se debe manejar el estado de "Eliminando" para evitar múltiples clics.
4. Tras una eliminación exitosa, es mandatorio navegar de vuelta a `ProductListScreen`.

---

## 2. DISEÑO

### API Endpoints Involucrados

| Método | Endpoint | Response |
|--------|----------|----------|
| `DELETE` | `/bp/products/:id` | `200: { message: "Product removed successfully" }` / `404: { error }` |

### Componente: `DeleteModal` (Diseño D4)

- **Ruta**: N/A (Componente dentro de `ProductDetailScreen`)
- **Elementos UI**:
  - Título o ícono de cierre (opcional)
  - Mensaje: `¿Estás seguro de eliminar el producto {productName}?`
  - Línea divisoria
  - Botón **"Confirmar"** (Estilo destacado)
  - Botón **"Cancelar"** (Estilo secundario)

#### Props del Componente
```typescript
interface DeleteModalProps {
  visible: boolean;
  productName: string;
  onConfirm: () => void;
  onCancel: () => void;
  isLoading?: boolean;
}
```

### Hook extendido: `useProducts`

```typescript
// Agregar a useProducts.ts:
deleteProduct: (id: string) => Promise<void>;
// Internamente: llama productService.deleteProduct -> refresca lista (opcional)
```

### Service

```typescript
// src/services/productService.ts
export async function deleteProduct(id: string): Promise<void>;
```

---

## 3. LISTA DE TAREAS

### Frontend Developer

- [ ] Crear `src/components/DeleteModal/DeleteModal.tsx` con estilos (`StyleSheet.create`)
  - Fondo semi-transparente (Overlay)
  - Contenedor blanco centrado
  - Botón Confirmar con `testID="btn-delete-confirm"`
  - Botón Cancelar con `testID="btn-delete-cancel"`
- [ ] Integrar `DeleteModal` en `src/screens/ProductDetailScreen/ProductDetailScreen.tsx`
  - Estado local `isModalVisible` para controlar el renderizado
  - Función `handleDelete` que llame a `deleteProduct(id)` y maneje la navegación tras éxito
- [ ] Extender `src/hooks/useProducts.ts` — agregar acción `deleteProduct`
- [ ] Agregar `deleteProduct` a `src/services/productService.ts` si no existe (usando `fetch` con `method: 'DELETE'`)

### Test Engineer Frontend

- [ ] `DeleteModal.test.tsx`:
  - Renderiza el nombre del producto correctamente
  - No es visible si `visible={false}`
  - Llama a `onConfirm` al presionar Confirmar
  - Llama a `onCancel` al presionar Cancelar
- [ ] `productService.test.ts` — deleteProduct:
  - Happy path: envía DELETE a la URL correcta, no lanza error
  - Error 404: lanza error con mensaje apropiado
- [ ] `useProducts.test.ts` — deleteProduct:
  - Remueve el producto de la lista local tras éxito (opcional si se recarga la lista)
- [ ] `ProductDetailScreen.test.tsx` (Eliminar):
  - Presionar botón "Eliminar" abre el modal
  - Confirmar en el modal llama a `deleteProduct` y navega a `ProductList`

### QA

- [ ] `/gherkin-case-generator` → CRITERIO-6.1 a 6.4
- [ ] `/risk-identifier`
- [ ] `/postman-collection-generator` → DELETE /bp/products/:id
- [ ] Verificar cobertura ≥ 70%
- [ ] Actualizar estado: `status: IMPLEMENTED`
