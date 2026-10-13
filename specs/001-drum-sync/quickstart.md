# Quickstart: validar Drum Sync de punta a punta

Guía para comprobar que la funcionalidad completa trabaja. No contiene código de implementación: los detalles están en `contracts/`, `data-model.md` y, más adelante, en `tasks.md`.

## Requisitos previos

- Node en versión LTS y npm.
- Docker, para el backend local de pruebas de Supabase (R-12 de `research.md`).
- Chrome o Edge para el modo de desarrollo en navegador.
- Una batería electrónica con salida USB MIDI, solo para la validación manual del Modo Práctica y la grabación reales. Los escenarios automáticos no la necesitan.

## Comandos (de AGENTS.md)

```bash
npm install
npm run dev        # Vite en http://localhost:5174
npx electron .     # en otra terminal, con Vite ya levantado
npm test
```

Comandos que el plan agrega y que crearán las tareas:

| Comando | Para qué |
|---|---|
| `npm run build:test` | compila el renderizador con la capacidad de pruebas activa, para los escenarios en Electron (la compilación de producción nunca la incluye) |
| `npm run test:e2e` | todos los escenarios Gherkin con Playwright, en navegador y en Electron |
| `npm run verify` | Vitest más todos los escenarios; es el paso previo a cada commit (constitución, principio II) |
| `npx supabase start` | backend local de pruebas con base, Auth y Storage |

## Escenarios de validación

Cada fila agrupa los AC del PRD que se prueban con Playwright. El resultado esperado es el de los escenarios del spec.

| # | Qué se valida | Historia del spec | AC principales | Cómo |
|---|---|---|---|---|
| 1 | Escribir con el mouse: posiciones, silencios, barras, tresillos, articulaciones, compases y Deshacer | 2 | AC-01 a AC-17, AC-117, AC-139, AC-140 | navegador, sin servidor |
| 2 | Cambio de tipo de compás y tempo | 3 | AC-40 a AC-44, AC-108, AC-109, AC-134, AC-135 | navegador |
| 3 | Reproducción, bucle, metrónomo y cuenta previa | 3 | AC-45 a AC-53, AC-85 | navegador con reloj controlado |
| 4 | Kit: carga, error y reintento | 3 | AC-54, AC-55, AC-114 | navegador con red simulada |
| 5 | Identificación, sesión única y mensajes | 1 | AC-74 a AC-82, AC-131, AC-136, AC-137 | Electron con usuarios de prueba |
| 6 | Autoguardado, biblioteca y confirmaciones | 4 | AC-56 a AC-64, AC-106, AC-110, AC-111, AC-116, AC-119, AC-123, AC-130 | navegador contra Supabase local |
| 7 | MIDI: conexión, mapeo, avisos y desconexión | 5 | AC-18 a AC-25, AC-90, AC-115 | navegador con MIDI simulado |
| 8 | Grabar tomas: cuantización, duración, articulación y fin | 5 | AC-26 a AC-39, AC-132, AC-133 | MIDI simulado y reloj controlado |
| 9 | Modo Práctica: colores, desvío, omitidas, bucle y volumen | 6 | AC-65 a AC-73, AC-112, AC-113, AC-118, AC-120, AC-121 | MIDI simulado y reloj controlado |
| 10 | Conexión: sin conexión, borrador, reconexión y conflicto | 7 | AC-83, AC-84, AC-105, AC-122, AC-124 a AC-127, AC-129 | Electron con red simulada |
| 11 | Calidad: rendimiento, formato, seguridad y plataformas | calidad | AC-79, AC-86 a AC-89, AC-91 a AC-104, AC-107, AC-128, AC-138 | escenarios Gherkin con Playwright: API directa, mediciones (`performance.feature`), instaladores (`installers.feature`) y Linux (`linux-keystore.feature`) |

## Antes de cada commit (constitución)

1. Ejecutar `npm run verify`. Si algún escenario falla, consultar a un humano antes de decidir si se deja pendiente.
2. Hacer el análisis OWASP Top 10 sobre los cambios y corregir las vulnerabilidades críticas y altas (escala OWASP Risk Rating).
3. Commit con mensaje en Conventional Commits.

## Antes de implementar la interfaz

Mostrar a un humano el diseño en imagen de las 7 pantallas listadas en el spec ("Pantallas y Estados") y esperar su aprobación (constitución, principio IV).

## Validación manual con batería real

Para lo que los escenarios no pueden comprobar sin hardware:
1. Conectar la batería, elegirla en el listado y golpear un pad: se ven nota, canal e intensidad.
2. Grabar una toma de 1 compás en grilla de semicorchea y revisar que las notas queden en su paso.
3. Activar el Modo Práctica con metrónomo y comprobar que los colores y los desvíos coinciden con lo tocado.
4. Objetivos de producto sin prueba automática: capturar un groove de 2 compases en menos de 2 minutos y que un baterista entienda un compás sin explicación (SC-002, SC-003).
