---
name: unit-testing
description: Genera tests unitarios con Jest + React Native Testing Library para el proyecto de prueba técnica. Coverage mínimo 70%. Lee la spec y el código implementado. Requiere spec APPROVED e implementación completa.
argument-hint: "<nombre-feature> [services|hooks|components|screens|all]"
---

# Unit Testing (React Native — Jest + RNTL)

## Definition of Done — verificar al completar

- [ ] Cobertura ≥ **70%** en líneas, funciones, branches y statements (quality gate bloqueante de la prueba técnica)
- [ ] Tests aislados — sin llamadas HTTP reales a `http://localhost:3002` (siempre mocks de fetch)
- [ ] Escenario feliz + errores de negocio + validaciones de entrada cubiertos
- [ ] Los cambios no rompen contratos existentes del módulo

## Prerequisito — Lee en paralelo

```
.github/specs/<feature>.spec.md                     (criterios de aceptación)
src/services/productService.ts                       (código implementado)
src/hooks/useProducts.ts                             (código implementado)
src/components/                                      (código implementado)
src/screens/                                         (código implementado)
.github/instructions/tests.instructions.md           (patrones Jest + RNTL)
```

## Output por scope

### Todos los tests → `src/__tests__/`

| Archivo | Capa | Qué cubre |
|---------|------|-----------|
| `services/productService.test.ts` | Services | GET, POST, PUT, DELETE, verificación ID, errores HTTP |
| `hooks/useProducts.test.ts` | Hooks | Estado inicial, loading, CRUD, errores, filtro búsqueda |
| `components/ProductCard.test.tsx` | Components | Render, onPress, props edge cases |
| `components/ProductForm.test.tsx` | Components | Validaciones campo a campo, submit, reset |
| `components/DeleteModal.test.tsx` | Components | Visible, confirmar, cancelar |
| `components/ProductList.test.tsx` | Components | Búsqueda, contador de registros |
| `screens/ProductListScreen.test.tsx` | Screens | Integración listado + búsqueda + nav |
| `screens/AddProductScreen.test.tsx` | Screens | F4: Formulario completo + validaciones |
| `screens/EditProductScreen.test.tsx` | Screens | F5: ID deshabilitado + datos precargados |

## Patrones Core

### Mock de fetch global (para servicios)

```typescript
// services/productService.test.ts
global.fetch = jest.fn();

beforeEach(() => jest.clearAllMocks());
afterEach(() => jest.restoreAllMocks());

it('GET /bp/products retorna lista de productos', async () => {
  const mockData = [{ id: 'p1', name: 'Producto 1', ... }];
  (global.fetch as jest.Mock).mockResolvedValueOnce({
    ok: true,
    json: async () => ({ data: mockData }),
  });

  const result = await getProducts();

  expect(result).toEqual(mockData);
  expect(global.fetch).toHaveBeenCalledWith('http://localhost:3002/bp/products');
});

it('GET /bp/products lanza error en status no-ok', async () => {
  (global.fetch as jest.Mock).mockResolvedValueOnce({ ok: false, status: 500 });
  await expect(getProducts()).rejects.toThrow('Error 500');
});
```

### Mock de servicios en hooks

```typescript
// hooks/useProducts.test.ts
jest.mock('../../services/productService');
import * as service from '../../services/productService';

const mockGetProducts = service.getProducts as jest.Mock;

it('carga productos en el mount', async () => {
  mockGetProducts.mockResolvedValueOnce([{ id: 'p1', name: 'Test', ... }]);

  const { result } = renderHook(() => useProducts());

  expect(result.current.loading).toBe(true);
  await waitFor(() => expect(result.current.loading).toBe(false));
  expect(result.current.products).toHaveLength(1);
});

it('maneja error de red correctamente', async () => {
  mockGetProducts.mockRejectedValueOnce(new Error('Network error'));

  const { result } = renderHook(() => useProducts());

  await waitFor(() => expect(result.current.loading).toBe(false));
  expect(result.current.error).toBe('Network error');
  expect(result.current.products).toHaveLength(0);
});
```

### Test de Componente con RNTL

```typescript
// components/ProductCard.test.tsx
import { render, fireEvent } from '@testing-library/react-native';

it('renderiza el nombre del producto', () => {
  const { getByText } = render(
    <ProductCard product={mockProduct} onPress={jest.fn()} />
  );
  expect(getByText(mockProduct.name)).toBeTruthy();
});

it('llama onPress al tocar la tarjeta', () => {
  const onPress = jest.fn();
  const { getByTestId } = render(
    <ProductCard product={mockProduct} onPress={onPress} testID="product-card" />
  );
  fireEvent.press(getByTestId('product-card'));
  expect(onPress).toHaveBeenCalledWith(mockProduct);
});
```

### Test de Validaciones del Formulario

```typescript
// components/ProductForm.test.tsx
it('muestra error cuando ID tiene menos de 3 caracteres', async () => {
  const { getByTestId, findByText } = render(<ProductForm onSubmit={jest.fn()} />);

  fireEvent.changeText(getByTestId('input-id'), 'ab');
  fireEvent.press(getByTestId('btn-submit'));

  expect(await findByText(/mínimo 3 caracteres/i)).toBeTruthy();
});

it('el botón Reiniciar limpia todos los campos', () => {
  const { getByTestId } = render(<ProductForm onSubmit={jest.fn()} />);

  fireEvent.changeText(getByTestId('input-name'), 'Mi Producto');
  fireEvent.press(getByTestId('btn-reset'));

  expect(getByTestId('input-name').props.value).toBe('');
});
```

### Mock de Navegación (React Navigation)

```typescript
// Mock estándar para React Navigation en tests
const mockNavigate = jest.fn();
const mockGoBack = jest.fn();

jest.mock('@react-navigation/native', () => ({
  ...jest.requireActual('@react-navigation/native'),
  useNavigation: () => ({ navigate: mockNavigate, goBack: mockGoBack }),
  useRoute: () => ({ params: { product: mockProduct } }),
}));
```

## Comando para verificar cobertura

```bash
# Ejecutar con reporte de cobertura
npx jest --coverage

# Con threshold estricto (igual al gate de la prueba técnica)
npx jest --coverage --coverageThreshold='{"global":{"lines":70,"functions":70,"branches":70,"statements":70}}'
```

## Restricciones

- Solo `src/__tests__/`. No modificar código fuente.
- Nunca llamar a `http://localhost:3002` en tests — siempre mockear `fetch`.
- Cobertura mínima ≥ **70%** en todas las métricas.
- Mock de navegación: `jest.fn()` para `navigate` y `goBack`.
- Preferir `getByText`, `getByLabelText` sobre `getByTestId` cuando sea posible.
