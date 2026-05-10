---
applyTo: "src/__tests__/**/*.{ts,tsx}"
---

> **Scope**: Reglas de testing para React Native con Jest + React Native Testing Library. Cobertura mínima **70%** (requerida por la prueba técnica).

# Instrucciones para Archivos de Pruebas — React Native (Jest + RNTL)

## Principios

- **Independencia**: Cada test es 100% independiente — sin estado compartido entre tests.
- **Aislamiento**: Mockear SIEMPRE servicios externos y llamadas HTTP.
- **Cobertura**: Cubrir happy path, error path y edge cases. **Cobertura mínima ≥ 70%** (gate bloqueante de la prueba técnica).
- **AAA**: Estructura Arrange → Act → Assert en todos los tests.

## Stack de Testing

| Herramienta | Uso |
|-------------|-----|
| **Jest** | Runner de tests, mocks, coverage |
| **@testing-library/react-native** | Render de componentes, queries, fireEvent |
| **@testing-library/jest-native** | Matchers adicionales para RN |
| `jest.fn()` / `jest.mock()` | Mockear módulos y funciones |

## Configuración Jest para React Native

```js
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterFramework: ['@testing-library/jest-native/extend-expect'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/navigation/**',  // Excluir navegación (difícil de testear)
    '!src/App.tsx',
  ],
  coverageThreshold: {
    global: {
      lines: 70,
      functions: 70,
      branches: 70,
      statements: 70,
    },
  },
};
```

## Estructura de Archivos de Tests

```
src/__tests__/
  services/
    productService.test.ts    ← Tests de llamadas HTTP (mockeando fetch)
  hooks/
    useProducts.test.ts       ← Estado inicial, CRUD, loading, errores
  components/
    ProductCard.test.tsx      ← Render, interacciones, props edge cases
    ProductForm.test.tsx      ← Validaciones campo a campo, submit, reset
    DeleteModal.test.tsx      ← Render condicional, confirmar, cancelar
    ProductList.test.tsx      ← Búsqueda, filtrado, contador de registros
  screens/
    ProductListScreen.test.tsx  ← Integración con hook y navegación
    AddProductScreen.test.tsx   ← Formulario + validaciones + submit
    EditProductScreen.test.tsx  ← ID deshabilitado + validaciones + submit
```

## Patrones Obligatorios

### Mockear servicios

```typescript
// En el test: SIEMPRE mockear servicios
jest.mock('../../services/productService');
import * as productService from '../../services/productService';

const mockGetProducts = productService.getProducts as jest.Mock;
```

### Test de Servicio (fetch mock)

```typescript
// services/productService.test.ts
import { getProducts, createProduct, verifyProductId } from '../../services/productService';

global.fetch = jest.fn();

beforeEach(() => {
  jest.clearAllMocks();
});

describe('productService', () => {
  describe('getProducts', () => {
    it('should return products on success', async () => {
      // ARRANGE
      const mockProducts = [{ id: 'trj-crd', name: 'Tarjeta de Crédito', ... }];
      (global.fetch as jest.Mock).mockResolvedValueOnce({
        ok: true,
        json: async () => ({ data: mockProducts }),
      });

      // ACT
      const result = await getProducts();

      // ASSERT
      expect(result).toEqual(mockProducts);
      expect(global.fetch).toHaveBeenCalledWith('http://localhost:3002/bp/products');
    });

    it('should throw error on non-ok response', async () => {
      (global.fetch as jest.Mock).mockResolvedValueOnce({ ok: false, status: 500 });
      await expect(getProducts()).rejects.toThrow('Error 500');
    });
  });

  describe('verifyProductId', () => {
    it('should return true when ID exists', async () => {
      (global.fetch as jest.Mock).mockResolvedValueOnce({
        ok: true,
        json: async () => true,
      });
      const result = await verifyProductId('trj-crd');
      expect(result).toBe(true);
    });
  });
});
```

### Test de Hook

```typescript
// hooks/useProducts.test.ts
import { renderHook, waitFor, act } from '@testing-library/react-native';
import { useProducts } from '../../hooks/useProducts';
import * as productService from '../../services/productService';

jest.mock('../../services/productService');

const mockProducts = [{ id: 'p1', name: 'Producto 1', ... }];

describe('useProducts', () => {
  beforeEach(() => jest.clearAllMocks());

  it('should load products on mount', async () => {
    (productService.getProducts as jest.Mock).mockResolvedValueOnce(mockProducts);
    const { result } = renderHook(() => useProducts());

    expect(result.current.loading).toBe(true);
    await waitFor(() => expect(result.current.loading).toBe(false));
    expect(result.current.products).toEqual(mockProducts);
  });

  it('should set error when getProducts fails', async () => {
    (productService.getProducts as jest.Mock).mockRejectedValueOnce(new Error('Network error'));
    const { result } = renderHook(() => useProducts());

    await waitFor(() => expect(result.current.loading).toBe(false));
    expect(result.current.error).toBe('Network error');
  });
});
```

### Test de Componente (React Native Testing Library)

```typescript
// components/ProductCard.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { ProductCard } from '../../components/ProductCard/ProductCard';

const mockProduct = {
  id: 'trj-crd',
  name: 'Tarjeta de Crédito',
  description: 'Tarjeta de consumo',
  logo: 'https://example.com/logo.png',
  date_release: '2025-01-01',
  date_revision: '2026-01-01',
};

describe('ProductCard', () => {
  it('renders product name correctly', () => {
    const { getByText } = render(
      <ProductCard product={mockProduct} onPress={jest.fn()} />
    );
    expect(getByText('Tarjeta de Crédito')).toBeTruthy();
  });

  it('calls onPress when card is pressed', () => {
    const onPress = jest.fn();
    const { getByTestId } = render(
      <ProductCard product={mockProduct} onPress={onPress} testID="product-card" />
    );
    fireEvent.press(getByTestId('product-card'));
    expect(onPress).toHaveBeenCalledWith(mockProduct);
  });
});
```

### Test de Formulario — Validaciones

```typescript
// components/ProductForm.test.tsx
describe('ProductForm validations', () => {
  it('shows error when ID is shorter than 3 chars', async () => {
    const { getByPlaceholderText, getByText, findByText } = render(
      <ProductForm onSubmit={jest.fn()} />
    );
    fireEvent.changeText(getByPlaceholderText('ID'), 'ab');
    fireEvent.press(getByText('Agregar'));
    expect(await findByText(/mínimo 3 caracteres/i)).toBeTruthy();
  });

  it('shows error when date_release is in the past', async () => {
    // ...
  });
});
```

### Test de Modal de Eliminación

```typescript
// components/DeleteModal.test.tsx
describe('DeleteModal', () => {
  it('calls onConfirm when Eliminar is pressed', () => {
    const onConfirm = jest.fn();
    const { getByText } = render(
      <DeleteModal visible={true} productName="Tarjeta" onConfirm={onConfirm} onCancel={jest.fn()} />
    );
    fireEvent.press(getByText('Eliminar'));
    expect(onConfirm).toHaveBeenCalled();
  });

  it('calls onCancel when Cancelar is pressed', () => {
    const onCancel = jest.fn();
    const { getByText } = render(
      <DeleteModal visible={true} productName="Tarjeta" onConfirm={jest.fn()} onCancel={onCancel} />
    );
    fireEvent.press(getByText('Cancelar'));
    expect(onCancel).toHaveBeenCalled();
  });
});
```

## Cobertura por Funcionalidad (F1–F6)

| Funcionalidad | Archivos de Test | Escenarios mínimos |
|---------------|-----------------|-------------------|
| F1 — Listado | `ProductListScreen.test.tsx`, `ProductCard.test.tsx` | Render lista, ítem vacío, carga |
| F2 — Búsqueda | `ProductList.test.tsx` | Filtrado por texto, sin resultados |
| F3 — Contador | `ProductList.test.tsx` | Muestra N registros correctos |
| F4 — Agregar | `AddProductScreen.test.tsx`, `ProductForm.test.tsx` | Validaciones todos los campos, submit exitoso |
| F5 — Editar | `EditProductScreen.test.tsx`, `ProductForm.test.tsx` | ID deshabilitado, validaciones, submit exitoso |
| F6 — Eliminar | `DeleteModal.test.tsx` | Confirmar elimina, cancelar oculta modal |

## Nunca hacer

- Tests que llamen a `http://localhost:3002` (siempre mockear `fetch`).
- Usar `getByTestId` para texto visible — preferir `getByText` o `getByLabelText`.
- Lógica de negocio compleja dentro de los tests.
- Omitir aserciones — cada `it()` debe tener al menos un `expect()`.

---

> Para criterios Gherkin avanzados, ver `.github/docs/lineamientos/qa-guidelines.md`.
