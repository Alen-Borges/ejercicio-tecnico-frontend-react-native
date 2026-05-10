---
name: generate-tests
description: Genera pruebas unitarias para backend (JUnit 5) y/o frontend (Vitest) en paralelo, basadas en la spec ASDD y el código implementado.
argument-hint: "<nombre-feature> [--backend] [--frontend] (por defecto genera ambos en paralelo)"
agent: Orchestrator
tools:
  - edit/createFile
  - edit/editFiles
  - read/readFile
  - search/listDirectory
  - search
  - execute/runInTerminal
---

# Task: Generate Backend Tests (JUnit 5 + Mockito)

Genera la suite de pruebas unitarias para la funcionalidad implementada.

## Lineamientos
- **Framework**: JUnit 5 + Mockito + AssertJ.
- **Estilo**: BDD (Behavior Driven Development) usando `given()`, `when()`, `then()`.
- **Aislamiento**: Mockear todas las dependencias.

## Estructura de Salida
1. **Service Tests**: Pruebas de lógica de negocio aisladas.
2. **Controller Tests**: Pruebas de endpoints usando `WebMvcTest` + `MockMvc`.
3. **Repository Tests (Opcional)**: Pruebas con `@DataJpaTest` si hay queries personalizadas.

## Ejemplo de Test (Service)
```java
@ExtendWith(MockitoExtension.class)
class MyServiceTest {
    @Mock private MyRepository repository;
    @InjectMocks private MyService service;

    @Test
    void shouldCreate_WhenValid() {
        // GIVEN
        given(repository.save(any())).willReturn(new MyEntity());
        // WHEN
        service.create(new MyDto());
        // THEN
        verify(repository).save(any());
    }
}
```

**Feature**: ${input:featureName:nombre del feature en kebab-case}
**Scope**: ${input:scope:backend, frontend, o ambos en paralelo (default)}

## Pasos obligatorios:

1. **Lee la spec** en `.github/specs/${input:featureName:nombre-feature}.spec.md` — sección "Plan de Pruebas Unitarias".
2. **Si scope es "ambos"**: lanza en paralelo `Test Engineer Backend` + `Test Engineer Frontend`.
3. **Si scope es "backend"**: delega a `Test Engineer Backend`:
   - `backend/tests/services/test_${input:featureName:feature}_service.py`
   - `backend/tests/repositories/test_${input:featureName:feature}_repository.py`
   - `backend/tests/routes/test_${input:featureName:feature}_router.py`
4. **Si scope es "frontend"**: delega a `Test Engineer Frontend`:
   - `frontend/src/__tests__/components/[Feature].test.jsx`
   - `frontend/src/__tests__/hooks/use[Feature].test.js`
   - `frontend/src/__tests__/pages/[Feature]Page.test.jsx`
5. **Verifica** que los tests corren:
   - Backend: `cd backend && poetry run pytest tests/ -v`
   - Frontend: `cd frontend && npx vitest run`

## Cobertura obligatoria por test:
- ✅ Happy path (flujo exitoso)
- ❌ Error path (excepciones, errores de red, datos inválidos)
- 🔲 Edge cases (campos vacíos, duplicados, permisos)

## Restricciones:
- Cada test debe ser independiente (no compartir estado).
- Mockear SIEMPRE las dependencias externas (DB, Firebase, API).
- Para backend: usar `pytest-asyncio` + `unittest.mock.AsyncMock`.
- Para frontend: usar `vitest` + `@testing-library/react`.
