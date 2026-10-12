# Implementation Plan: Drum Sync — Escribir, escuchar y practicar batería

**Branch**: `001-drum-sync` | **Date**: 2026-10-12 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-drum-sync/spec.md`

**Fuentes de contexto técnico**: `Claude.md` (que solo importa `AGENTS.md`), `AGENTS.md` (stack, comandos y prohibiciones), `.specify/memory/constitution.md` v1.0.1 y `PRD.md`.

## Summary

Drum Sync es un editor de partitura de batería que suena y sirve para practicar con una batería MIDI. El plan lo construye como una **aplicación de escritorio Electron** con cliente React + TypeScript sobre Vite, notación con VexFlow, y **Supabase** como único backend (Auth con Google, Postgres con Row Level Security y Storage para el kit), sin servidor propio.

El enfoque técnico separa un **núcleo en TypeScript puro** (grilla de 48 unidades por negra, modelo de partitura, cuantización, silencios, evaluación de la práctica, migraciones de formato) de las capas de audio, MIDI, notación, datos y UI. Así el núcleo se prueba sin servidor ni Electron, y todo lo que afecta al timing (Web Audio y Web MIDI) corre localmente con sonidos ya decodificados en memoria. Cada AC del PRD se convierte en un escenario Gherkin automatizado con **Playwright**, apoyado en una capacidad de pruebas que inyecta golpes MIDI simulados y controla el reloj de audio (RNF-18).

## Technical Context

**Language/Version**: TypeScript en modo estricto, sobre Node en versión LTS. Las versiones exactas las fija `package.json` (AGENTS.md).

**Primary Dependencies**: React, Vite, VexFlow, `supabase-js`, Electron (con Electron Forge solo para empaquetar), Vitest, Playwright con `playwright-bdd` para ejecutar archivos `.feature` (ver research R-01).

**Storage**: Postgres de Supabase como única base de datos (perfiles, biblioteca, partitura en curso, configuración) y Supabase Storage para el kit. En la computadora solo se guardan, cifrados con `safeStorage`, la sesión y el borrador sin conexión (AGENTS.md, RNF-13). No hay base local.

**Testing**: Vitest para el núcleo (`tests/unit`) y Playwright para los escenarios Gherkin (`tests/features`), ejecutados contra el cliente en Chrome o Edge en localhost y contra Electron. Servidor de pruebas local con la CLI de Supabase (R-12).

**Target Platform**: Windows, macOS y Linux como aplicación Electron (RNF-17). En desarrollo el cliente también corre en Chrome o Edge en `http://localhost:5174`.

**Project Type**: desktop-app (Electron) con backend gestionado (BaaS), sin servidor propio.

**Performance Goals**: eventos de audio a menos de 1 ms de su instante teórico (RNF-01); golpe visible en menos de 250 ms p95 y feedback de práctica en menos de 100 ms p95 con desvío fiel a ±20 ms (RNF-02, RNF-03); partitura en curso en menos de 3 s p95 y kit decodificado en 5 s o menos p95 con 10 Mbps (RNF-08, RNF-15).

**Constraints**: la red nunca está en el camino del timing (RNF-15); el editor y la reproducción no dependen de Web MIDI (RNF-07); kit de 3 MB o menos; autoguardado a 5 s como máximo (RNF-16); sin `service_role` en la app (RNF-14); solo sesión y borrador cifrados en disco (RNF-13).

**Scale/Scope**: hasta 128 compases por partitura, 13 piezas del kit, 7 pantallas, 138 AC del PRD, número de partituras por usuario sin límite.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principio | Cómo lo cumple el plan | Estado |
|---|---|---|
| I. Escenarios Gherkin automatizados con Playwright | Cada AC del PRD es un escenario `.feature` ejecutado con Playwright + `playwright-bdd`; trazabilidad por ID de AC en la etiqueta de cada escenario. La capacidad de pruebas RNF-18 hace posible automatizar golpes MIDI y reloj. | Cumple |
| II. Commit solo con la suite completa en verde | Un script `npm run verify` ejecuta Vitest y todos los escenarios Playwright; es el paso previo a cada commit. Si falla algún escenario se consulta a un humano. | Cumple |
| III. Análisis OWASP Top 10 antes de commitear | Paso `security-review` en el flujo de cada commit con la escala OWASP Risk Rating; críticas y altas se corrigen antes. El diseño aplica RLS, sesión cifrada y sin `service_role`. | Cumple |
| IV. Diseño visual aprobado antes de implementar | Se agrega una **Fase de diseño visual** antes de cualquier tarea de interfaz: imágenes de las 7 pantallas y aprobación humana. Las tareas de UI quedan bloqueadas hasta entonces. | Cumple, con compuerta |
| AGENTS.md: no romper partituras guardadas | Formato versionado (`formatVersion`) y módulo de migraciones con pruebas por versión (RNF-09, RNF-09.1). | Cumple |
| AGENTS.md: editor y reproducción sin Web MIDI | Capa `midi` aislada; fallo o permiso denegado solo deshabilita grabación y práctica. | Cumple |
| AGENTS.md: sin red en el camino del timing | Kit decodificado en `AudioBuffer` antes de habilitar Play; el scheduler no hace ningún pedido. | Cumple |
| AGENTS.md: RLS, sin `service_role` | Toda tabla con RLS y `owner_id = auth.uid()`; la app solo lleva la clave pública. | Cumple |
| AGENTS.md: solo sesión y borrador cifrados en disco | `safeStorage` en el proceso principal de Electron; en navegador de desarrollo no se persiste nada. | Cumple |

**Resultado de la compuerta inicial**: sin violaciones. No hay entradas en Complexity Tracking.

**Re-evaluación posterior al diseño (Phase 1)**: se mantiene sin violaciones. Las dudas técnicas quedaron resueltas en `research.md`: sesión única (R-10, ajuste de Supabase más comprobación del `session_id` en RLS), almacén seguro en Linux (R-09, verificado), equipo de referencia para los p95 (la computadora de desarrollo) y versiones fijas (R-16). Quedan dos validaciones prácticas para las tareas: el spike de R-10 y la prueba en Linux de R-09.

## Project Structure

### Documentation (this feature)

```text
specs/001-drum-sync/
├── plan.md              # Este archivo (/speckit-plan)
├── research.md          # Phase 0: decisiones y alternativas
├── data-model.md        # Phase 1: entidades, validaciones y estados
├── quickstart.md        # Phase 1: guía de validación de punta a punta
├── contracts/           # Phase 1: interfaces del proyecto
│   ├── database.md      #   tablas, políticas RLS y Storage de Supabase
│   ├── ipc-preload.md   #   puente seguro entre Electron y el cliente
│   ├── score-format-v1.md #   formato de partitura versión 1
│   └── test-seam.md     #   capacidad de pruebas (RNF-18)
├── checklists/          # requirements.md y general.md
└── tasks.md             # Phase 2 (/speckit-tasks, no lo crea este comando)
```

### Source Code (repository root)

```text
src/
├── core/                # TypeScript puro, sin Electron ni Supabase
│   ├── grid.ts          #   48 unidades por negra y tablas de figuras
│   ├── score.ts         #   modelo de partitura, compases y notas
│   ├── rests.ts         #   silencios derivados por voz
│   ├── timesig.ts       #   cambio de tipo de compás y tope de 128
│   ├── quantizer.ts     #   cuantización, fusión y duración de toma
│   ├── articulation.ts  #   articulación por intensidad y umbrales
│   ├── practice.ts      #   comparación, desvío, colores y omitidas
│   ├── undo.ts          #   historial de 100 cambios
│   ├── library.ts       #   nombres, unicidad y cambios sin guardar
│   └── migrations/      #   migraciones del formato por versión
├── audio/               # Web Audio: scheduler, metrónomo, kit en memoria
├── midi/                # Web MIDI, mapeo General MIDI (midi.ts), captura de golpes
├── notation/            # Render VexFlow y detección de clic (pieza y posición)
├── data/                # supabase-js, sesión, repositorios, borrador y conexión
├── ui/                  # React: pantallas, componentes y estado
│   ├── screens/         #   identificación, editor, biblioteca, práctica
│   └── components/
├── test-seam/           # inyección de golpes y reloj (solo entorno de pruebas)
└── main.tsx

electron/
├── main.ts              # ventana, permisos MIDI, protocolo drumsync://
├── preload.ts           # API segura expuesta al cliente
└── secureStore.ts       # sesión y borrador cifrados con safeStorage

supabase/
├── migrations/          # tablas, RLS, función de sesión activa, disparador de perfil
└── seed/                # datos de prueba y manifiesto del kit

tests/
├── features/            # escenarios Gherkin (.feature), uno por grupo de AC
├── steps/               # definiciones de pasos de Playwright
├── support/             # fixtures, servidor local de Supabase, usuarios de prueba
└── unit/                # Vitest del núcleo
```

**Structure Decision**: proyecto único de escritorio (`src/` más `electron/`), con `supabase/` para el backend y `tests/` para las dos suites. `src/core` no importa nada de `data`, `audio`, `midi`, `ui` ni Electron, de modo que el editor, la reproducción, la grabación y el Modo Práctica se desarrollan y prueban sin servidor (riesgo señalado en el PRD).

## Orden de entrega propuesto

Insumo para `/speckit-tasks`, por prioridad de las historias del spec:

1. **Base**: estructura del proyecto, `core` con grilla y modelo de partitura, `tests` con Vitest y Playwright, `verify`.
2. **Historia 2 (editor con mouse)**: `core`, `notation`. La interfaz se bloquea hasta aprobar el diseño visual.
3. **Historia 3 (escuchar)**: `audio`, kit y metrónomo; carga del kit desde el servidor.
4. **Historia 1 (identificación)** y **Historia 4 (guardado y biblioteca)**: `data`, `supabase`, `electron`.
5. **Historia 5 (grabar tomas)** y **Historia 6 (Modo Práctica)**: `midi`, `core`.
6. **Historia 7 (conexión)** y escenarios de calidad (rendimiento, seguridad, plataformas).

La historia 1 es P1 en el spec pero depende de Electron y Supabase; el plan construye antes el núcleo y el editor, que no la necesitan, para validar el timing desde el principio.

## Complexity Tracking

Sin violaciones de la constitución: no corresponde justificar excepciones.
