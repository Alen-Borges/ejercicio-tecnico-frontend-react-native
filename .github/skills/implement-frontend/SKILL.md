---
name: implement-frontend
description: Implementa un feature completo en React Native + TypeScript. Requiere spec con status APPROVED en .github/specs/.
argument-hint: "<nombre-feature> [F1|F2|F3|F4|F5|F6]"
---

# Implement Frontend (React Native)

## Prerequisitos

1. Leer spec: `.github/specs/<feature>.spec.md` — sección 2 (componentes, screens, hooks, services)
2. Leer stack y arquitectura: `.github/instructions/frontend.instructions.md`
3. Revisar código existente: explorar `src/` para NO duplicar componentes o lógica

## Orden de implementación

```
services → hooks/state → components → screens/views → registrar ruta en AppNavigator
```

| Capa | Responsabilidad | Archivo destino |
|------|-----------------|----------------|
| **Services** | Llamadas HTTP al backend local (fetch) — sin estado | `src/services/productService.ts` |
| **Hooks** | Estado, loading, errores, acciones CRUD — consume services | `src/hooks/useProducts.ts` |
| **Components** | UI reutilizable — props tipadas + eventos | `src/components/<ComponentName>/` |
| **Screens** | Composición final + layout + navegación | `src/screens/<ScreenName>/` |

## Patrones Obligatorios

### Tipado TypeScript

```typescript
// Modelo de dominio principal
interface FinancialProduct {
  id: string;
  name: string;
  description: string;
  logo: string;
  date_release: string;
  date_revision: string;
}

// Props tipadas (nunca any)
interface ProductCardProps {
  product: FinancialProduct;
  onPress: (product: FinancialProduct) => void;
  testID?: string;
}
```

### Service Pattern (fetch nativo)

```typescript
// src/services/productService.ts
const BASE_URL = 'http://localhost:3002';

export async function getProducts(): Promise<FinancialProduct[]> {
  const res = await fetch(`${BASE_URL}/bp/products`);
  if (!res.ok) throw new Error(`Error ${res.status}: ${res.statusText}`);
  const json = await res.json();
  return json.data;
}

export async function verifyProductId(id: string): Promise<boolean> {
  const res = await fetch(`${BASE_URL}/bp/products/verification/${id}`);
  if (!res.ok) throw new Error(`Error ${res.status}`);
  return res.json();
}
```

### Hook Pattern

```typescript
// src/hooks/useProducts.ts
export function useProducts() {
  const [products, setProducts] = useState<FinancialProduct[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const loadProducts = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const data = await getProducts();
      setProducts(data);
    } catch (e) {
      setError(e instanceof Error ? e.message : 'Error al cargar productos');
    } finally {
      setLoading(false);
    }
  }, []);

  return { products, loading, error, loadProducts };
}
```

### Estilos (StyleSheet.create — SIN librerías externas)

```typescript
const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  card: { padding: 16, marginVertical: 8, borderRadius: 8, elevation: 2 },
  errorText: { color: '#dc3545', fontSize: 12, marginTop: 4 },
});
```

### Validaciones del Formulario (F4 / F5)

```typescript
// Función de validación pura (fácil de testear)
export function validateProduct(
  data: Partial<FinancialProduct>,
  isEdit = false
): Record<string, string> {
  const errors: Record<string, string> = {};

  if (!data.id && !isEdit) errors.id = 'El ID es requerido';
  else if (!isEdit && data.id && data.id.length < 3) errors.id = 'Mínimo 3 caracteres';
  else if (!isEdit && data.id && data.id.length > 10) errors.id = 'Máximo 10 caracteres';

  if (!data.name) errors.name = 'El nombre es requerido';
  else if (data.name.length < 5) errors.name = 'Mínimo 5 caracteres';
  else if (data.name.length > 100) errors.name = 'Máximo 100 caracteres';

  if (!data.description) errors.description = 'La descripción es requerida';
  else if (data.description.length < 10) errors.description = 'Mínimo 10 caracteres';
  else if (data.description.length > 200) errors.description = 'Máximo 200 caracteres';

  if (!data.logo) errors.logo = 'El logo es requerido';

  const today = new Date().toISOString().split('T')[0];
  if (!data.date_release) errors.date_release = 'La fecha de liberación es requerida';
  else if (data.date_release < today) errors.date_release = 'Debe ser igual o mayor a hoy';

  if (!data.date_revision) errors.date_revision = 'La fecha de revisión es requerida';
  else if (data.date_release) {
    const expected = new Date(data.date_release);
    expected.setFullYear(expected.getFullYear() + 1);
    const expectedStr = expected.toISOString().split('T')[0];
    if (data.date_revision !== expectedStr)
      errors.date_revision = 'Debe ser exactamente 1 año después de la fecha de liberación';
  }

  return errors;
}
```

### Navegación Tipada (React Navigation v6)

```typescript
// src/navigation/AppNavigator.tsx
export type RootStackParamList = {
  ProductList: undefined;
  ProductDetail: { product: FinancialProduct };
  AddProduct: undefined;
  EditProduct: { product: FinancialProduct };
};

// En screen: uso tipado
const navigation = useNavigation<NativeStackNavigationProp<RootStackParamList>>();
const { product } = useRoute<RouteProp<RootStackParamList, 'EditProduct'>>().params;
```

### testID para Testing

```tsx
// Agregar testID a elementos interactivos clave
<TouchableOpacity testID="product-card" onPress={onPress} />
<TextInput testID="input-id" value={id} onChangeText={setId} />
<TouchableOpacity testID="btn-submit" onPress={handleSubmit} />
<TouchableOpacity testID="btn-reset" onPress={handleReset} />
<TouchableOpacity testID="btn-delete-confirm" onPress={onConfirm} />
<TouchableOpacity testID="btn-delete-cancel" onPress={onCancel} />
```

## Restricciones

- Solo directorio `src/` del proyecto. No tocar `android/`, `ios/`, ni config nativa.
- No generar tests (responsabilidad de `test-engineer-frontend`).
- **SIN** librerías de UI externas (NativeBase, Paper, Tamagui, Tailwind RN, etc.).
- **SIN** `any` en TypeScript.
- Mostrar SIEMPRE errores visuales al usuario.
- Seguir los lineamientos en `.github/instructions/frontend.instructions.md`.
