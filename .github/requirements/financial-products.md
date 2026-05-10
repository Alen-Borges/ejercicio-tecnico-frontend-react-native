# Requerimiento: Prueba Técnica Frontend — Productos Financieros (2024)

**Origen**: Prueba técnica oficial — perfil Senior  
**Stack**: React Native 0.73+ / TypeScript 4.8+ / Jest  
**Backend local**: `http://localhost:3002` (Node.js)

---

## Funcionalidades Requeridas

### F1 — Listado de Productos Financieros

Se requiere una aplicación para visualizar los diferentes productos financieros ofertados por un Banco cargados de una API.

- Cargar productos desde `GET http://localhost:3002/bp/products`
- Maquetación basada en **Diseño D1**
- Al seleccionar un ítem → navegar a la vista de detalle del producto

### F2 — Búsqueda de Productos Financieros

Se requiere realizar búsqueda de los productos financieros mediante un campo de texto.

- Campo `TextInput` de búsqueda en la misma pantalla del listado
- Filtrado en tiempo real por nombre o descripción
- Maquetación basada en **Diseño D1**

### F3 — Cantidad de Registros

Se requiere mostrar la cantidad de registros obtenidos (post-filtro).

- Texto que muestra `N resultados` debajo o encima del listado
- Maquetación basada en **Diseño D1**

### F4 — Agregar Producto

Se requiere la implementación de un botón "Agregar" para navegar al formulario de registro.

- Botón "Agregar" en la pantalla de listado (Diseño D3)
- Formulario con campos: ID, Nombre, Descripción, Logo, Fecha Liberación, Fecha Revisión
- Botón "Agregar" para enviar el formulario
- Botón "Reiniciar" para limpiar todos los campos
- Maquetación del formulario basada en **Diseño D2**

#### Validaciones de Campos (F4):

| Campo | Validación |
|-------|-----------|
| **ID** | Requerido, mínimo 3 caracteres, máximo 10, verificar que no exista via `GET /bp/products/verification/:id` |
| **Nombre** | Requerido, mínimo 5 caracteres, máximo 100 |
| **Descripción** | Requerido, mínimo 10 caracteres, máximo 200 |
| **Logo** | Requerido |
| **Fecha de Liberación** | Requerido, debe ser igual o mayor a la fecha actual |
| **Fecha de Revisión** | Requerido, debe ser exactamente un año posterior a la Fecha de Liberación |

Los errores de validación se muestran visualmente bajo cada campo.

### F5 — Editar Producto

Se requiere un botón que, al hacer clic, permita editar el producto.

- Navegar a la pantalla de edición del producto
- El campo ID debe estar **deshabilitado** (no editable)
- Pre-cargar los datos actuales del producto en el formulario
- Mismas validaciones que F4 (excepto verificación de ID único)
- Maquetación basada en **Diseño D2**

### F6 — Eliminar Producto

Se requiere un botón que, al hacer clic, muestre un modal de confirmación.

- Modal con texto: `¿Estás seguro de eliminar el producto {nombre}?`
- Botón **"Eliminar"** / **"Confirmar"** → ejecuta `DELETE /bp/products/:id`
- Botón **"Cancelar"** → solo oculta el modal
- Maquetación basada en **Diseño D4**

---

## API del Backend Local

**URL Base**: `http://localhost:3002`

| Método | Endpoint | Descripción | Response 200 |
|--------|----------|-------------|-------------|
| `GET` | `/bp/products` | Obtener todos los productos | `{ data: FinancialProduct[] }` |
| `POST` | `/bp/products` | Crear nuevo producto | `{ message, data: FinancialProduct }` |
| `PUT` | `/bp/products/:id` | Actualizar producto | `{ message, data: FinancialProduct }` |
| `DELETE` | `/bp/products/:id` | Eliminar producto | `{ message: string }` |
| `GET` | `/bp/products/verification/:id` | Verificar si ID existe | `true` o `false` |

**Errores**:
- `400` — Body inválido (POST/PUT)
- `404` — Producto no encontrado (PUT/DELETE)

---

## Modelo de Datos — `FinancialProduct`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | `string` | Identificador único (`trj-crd`) |
| `name` | `string` | Nombre del producto |
| `description` | `string` | Descripción del producto |
| `logo` | `string` | URL del logo representativo |
| `date_release` | `string (date)` | Fecha de liberación (`YYYY-MM-DD`) |
| `date_revision` | `string (date)` | Fecha de revisión (`YYYY-MM-DD`) |

---

## Criterios de Calidad

- **SOLID + Clean Code** — separación de responsabilidades entre capas
- **Sin frameworks de UI** — solo `StyleSheet.create()` de React Native
- **Manejo de errores** — mensajes visuales siempre al usuario
- **Pruebas unitarias** con Jest — cobertura mínima **70%**
- **Skeletons** de precarga (deseable para perfil Senior)

---

## Cómo generar la spec

```
/generate-spec financial-products
```

O desde el orchestrator:

```
/asdd-orchestrate
> Feature: product-list (F1 + F2 + F3)
```
