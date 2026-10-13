---

description: "Lista de tareas de Drum Sync"
---

# Tasks: Drum Sync — Escribir, escuchar y practicar batería

**Input**: Documentos de diseño en `/specs/001-drum-sync/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: **Obligatorios.** La constitución (principios I y II) exige que cada comportamiento tenga escenarios Gherkin automatizados con Playwright y que la suite completa esté en verde antes de cada commit. Por eso cada historia empieza con sus escenarios y sus pasos, que deben **fallar** antes de implementar. Los escenarios llevan la etiqueta de su AC (por ejemplo `@AC-66`).

**Organization**: Tareas agrupadas por historia de usuario del spec. Las fases siguen el orden de entrega del plan: núcleo y editor primero (no dependen de Electron ni de Supabase), luego sonido, identificación, biblioteca, MIDI, práctica y conexión.

## Format: `[ID] [P?] [Story] Descripción con ruta`

- **[P]**: se puede hacer en paralelo (archivos distintos, sin dependencias pendientes)
- **[Story]**: historia a la que pertenece (US1 a US7)
- Las tareas marcadas **🎨** son de interfaz y se apoyan en el diseño visual aprobado (T021, constitución, principio IV). Como T021 forma parte de Foundational, el diseño queda aprobado antes de empezar cualquier historia

## Cómo se prueba cada commit (constitución)

Antes de cada commit: `npm run verify` (Vitest más todos los escenarios Playwright) y análisis OWASP Top 10 sobre los cambios con la escala OWASP Risk Rating. Críticas y altas se corrigen antes del commit. Si un escenario falla se consulta a un humano.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Inicialización del proyecto y herramientas

- [ ] T001 Inicializar el proyecto npm en la raíz con `package.json`: instalar las versiones estables más recientes de React, Vite, VexFlow, `supabase-js`, Electron (igual o posterior a 25.5.0), Electron Forge, Vitest y Playwright con `playwright-bdd`, y fijarlas exactas; subir `package-lock.json` (research R-16)
- [ ] T002 [P] Crear `tsconfig.json` en modo estricto y `vite.config.ts` con el servidor en el puerto 5174 y una variable de entorno `VITE_TEST_SEAM` que activa la capacidad de pruebas
- [ ] T003 Crear la estructura de carpetas del plan: `src/core`, `src/audio`, `src/midi`, `src/notation`, `src/data`, `src/ui/screens`, `src/ui/components`, `src/test-seam`, `electron/`, `supabase/migrations`, `supabase/seed`, `tests/features`, `tests/steps`, `tests/support`, `tests/unit`
- [ ] T004 [P] Crear `.gitignore` (carpeta `node_modules`, `dist`, `.env`, resultados de Playwright, datos locales de Supabase) y `.env.example` solo con `VITE_SUPABASE_URL` y `VITE_SUPABASE_ANON_KEY`; la clave `service_role` nunca va en la app ni en `.env.example` (RNF-14)
- [ ] T005 [P] Configurar `vitest.config.ts` para `tests/unit` y comprobar que `npm test` corre
- [ ] T006 [P] Configurar `playwright.config.ts` con `playwright-bdd` (features en `tests/features`, pasos en `tests/steps`) y dos proyectos base: navegador contra `http://localhost:5174` y Electron (research R-02); el proyecto de Electron se ejecuta además en cada sistema operativo (Windows, macOS y Linux) para los escenarios que dependen del sistema (AC-79, AC-107). El proyecto de Electron lanza la aplicación con la compilación de pruebas (`npm run build:test`, T009), nunca con la de producción
- [ ] T007 [P] Crear el esqueleto de Electron: `electron/main.ts`, `electron/preload.ts` y `forge.config.ts` (Forge solo para empaquetar), con aislamiento de contexto activado y sin integración de Node en el renderizador
- [ ] T008 [P] Inicializar Supabase local con `npx supabase init` (`supabase/config.toml`) y documentar `npx supabase start` en `specs/001-drum-sync/quickstart.md`
- [ ] T009 Agregar a `package.json` los scripts `dev`, `build`, `build:test`, `test`, `test:e2e` y `verify` (Vitest más todos los escenarios en ambos proyectos), según `quickstart.md`. `build:test` compila el renderizador con `VITE_TEST_SEAM` activa para las pruebas en Electron; `build` (producción) nunca incluye la capacidad de pruebas (AC-128)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Núcleo común y capacidad de pruebas que todas las historias necesitan

**⚠️ CRITICAL**: Ninguna historia puede empezar hasta terminar esta fase

- [ ] T010 [P] Escribir primero `tests/unit/grid.test.ts` (AC-88): negra de 48 unidades; semicorchea 12, corchea 24, corchea de tresillo 16, blanca 96, redonda 192; duración de compás `numerador × (192 / denominador)`
- [ ] T011 Implementar `src/core/grid.ts` con las constantes de la grilla y `measureLength(numerator, denominator)` (depende de T010)
- [ ] T012 [P] Implementar los tipos y la validación de `src/core/score.ts` según `contracts/score-format-v1.md`: `tempo` entero de 30 a 260 (un valor fuera de rango se ajusta al límite más cercano), `timeSignature` con numerador de 1 a 16 y denominador 2, 4, 8 o 16, `measures` de 1 a 128 compases (nunca vacío), `notes[].piece` en los 13 valores del contrato, `start` y `duration` en unidades (la nota debe caber en el compás), y dos notas de la misma pieza no pueden compartir `start`
- [ ] T013 [P] Implementar `src/core/migrations/index.ts` (lleva cualquier versión anterior a la actual en orden, hoy solo existe la versión 1) y la partitura de prueba `tests/support/fixtures/score-v1.json` con notas en todas las figuras y piezas (RNF-09)
- [ ] T014 [P] Escribir `tests/unit/undo.test.ts` (AC-117) e implementar `src/core/undo.ts`: instantáneas inmutables, máximo 100 (se descarta la más antigua al superarlo), se pierde al cerrar la aplicación; una toma o un cambio de tipo de compás cuenta como un solo cambio
- [ ] T015 [P] Implementar `src/audio/clock.ts`: envoltorio del reloj de audio (`AudioContext.currentTime`) con una versión controlable para pruebas (research R-04)
- [ ] T016 Implementar `src/test-seam/index.ts` según `contracts/test-seam.md`, solo la parte de reloj y red: `clock.freeze/advance/scheduled` y `network.set/requests`; se incluye solo si `VITE_TEST_SEAM` está activa y nunca en producción. La parte de MIDI simulado se agrega en US5 (T119) porque necesita los módulos MIDI (depende de T015)
- [ ] T017 [P] Implementar `tests/support/supabase.ts`: levantar y reiniciar el servidor local de Supabase, crear usuarios de prueba y cargar el kit de prueba; la clave `service_role` se usa solo aquí, nunca en `src/` ni `electron/` (research R-12)
- [ ] T018 [P] Implementar `src/data/supabase.ts`: cliente `supabase-js` solo con la clave pública, flujo con PKCE y sin persistir la sesión fuera del puente seguro (RNF-13)
- [ ] T019 [P] Definir los pasos comunes de Playwright en `tests/steps/common.steps.ts`: abrir la aplicación, avanzar el reloj y simular la red. Los pasos de golpes MIDI van en US5 (T079) y los de inicio de sesión con un usuario de prueba en US1 (T060)
- [ ] T020 [P] Fijar en `tests/support/network.ts` cómo se limita la red a 10 Mbps para las mediciones de AC-91 y AC-104; el equipo de referencia para los p95 ya está registrado en `specs/001-drum-sync/research.md`
- [ ] T021 **🎨 Compuerta de diseño visual**: preparar maquetas en HTML de las 7 pantallas de la sección "Pantallas y Estados" del spec (identificación, editor, conexión de la batería, grabación de toma, Modo Práctica, biblioteca y diálogos), tomar capturas en PNG, guardarlas en `specs/001-drum-sync/design/`, mostrarlas a un humano, iterar con sus comentarios y registrar su aprobación en `specs/001-drum-sync/design/APROBADO.md`. Si el diseño cambia durante la implementación, se vuelve a mostrar (constitución, principio IV)

**Checkpoint**: Núcleo base, capacidad de pruebas y **diseño visual aprobado (T021)** listos. Por decisión del proyecto, el diseño se aprueba antes de empezar cualquier historia: ninguna historia comienza hasta cerrar esta fase completa.

---

## Phase 3: User Story 2 - Escribir una partitura con el mouse (Priority: P1) 🎯 MVP

**Goal**: Escribir compases de batería con el mouse en un pentagrama de percusión, con silencios, barras, tresillos, articulaciones, compases y Deshacer.

**Independent Test**: Sin servidor ni MIDI, escribir un compás de 4/4 con bombo, caja y hi-hat; comprobar posiciones, silencios, barras y voces; añadir y quitar compases; deshacer.

### Tests for User Story 2 ⚠️

> Escribirlos primero y comprobar que fallan antes de implementar

- [ ] T022 [P] [US2] Escribir `tests/features/editor.feature` con los escenarios de AC-01, AC-03, AC-04, AC-05, AC-06, AC-07, AC-08, AC-09, AC-10, AC-11, AC-12, AC-13, AC-14, AC-15, AC-17, AC-117, AC-139 y AC-140, cada uno con su etiqueta
- [ ] T023 [P] [US2] Escribir los pasos en `tests/steps/editor.steps.ts`
- [ ] T024 [P] [US2] Escribir `tests/unit/rests.test.ts` (AC-07, AC-08, AC-09, AC-89): en cada voz la suma de figuras y silencios es exactamente la duración del compás, incluidos tresillos con semicorcheas y compases de una toma
- [ ] T025 [P] [US2] Escribir `tests/unit/measures.test.ts` (AC-12 a AC-15): añadir compás hasta 128, quitar el último sin dejar la partitura sin compases; y `tests/unit/edit.test.ts` (AC-139, AC-140): seleccionar una nota no la borra ni escribe otra, y borrar la seleccionada recalcula los silencios y se deshace

### Implementation for User Story 2

- [ ] T026 [P] [US2] Implementar `src/core/rests.ts`: silencios derivados por voz (manos: todas las piezas salvo bombo y hi-hat con pie, plicas arriba; pies: bombo e hi-hat con pie, plicas abajo); un compás vacío tiene un único silencio de compás completo; los silencios no se guardan
- [ ] T027 [P] [US2] Implementar `src/core/measures.ts`: añadir compás (máximo 128) y quitar el último (nunca menos de 1)
- [ ] T028 [US2] Implementar `src/core/edit.ts`: escribir una nota (pieza, `start`, figura, articulación), con tresillo (tres figuras en el espacio de dos: corchea de tresillo de 16 unidades, `groupId` compartido) y rechazo de dos notas de la misma pieza en el mismo `start`; incluye `removeNote(id)` para borrar la nota seleccionada (RF-07.4) (depende de T012)
- [ ] T029 [P] [US2] Implementar `src/notation/staff.ts`: tabla de AC-03 que da, para cada una de las 13 piezas, su posición en el pentagrama y la forma de su cabeza (ovalada, cruz, cruz con "o", rombo)
- [ ] T030 [US2] Implementar `src/notation/render.ts` con VexFlow: pentagrama de percusión con clave neutra, dos voces, barras por tiempo, corchete y número 3 en tresillos, silencios y las marcas de articulación (">" para acento, cabeza entre paréntesis para fantasma, nota de adorno para flam) (depende de T026 y T029)
- [ ] T031 [US2] Implementar `src/notation/hitTest.ts`: convierte el clic en pieza (por la altura) y posición en la grilla (por la posición horizontal) (RF-07) y detecta si el clic cae sobre una nota ya escrita para seleccionarla en lugar de escribir otra (RF-07.3)
- [ ] T032 [US2] Implementar `src/ui/state/scoreStore.ts` que integra `core` con Deshacer (T014), la nota seleccionada y el borrado como un cambio de Deshacer sin confirmación; el editor no depende de Web MIDI ni de la red (RNF-07)
- [ ] T033 [US2] 🎨 Implementar `src/ui/screens/Editor.tsx` con el pentagrama, el selector de figura (redonda, blanca, negra, corchea, semicorchea) con tresillo, el selector de articulación (normal, acento, fantasma, flam), selección de nota con clic, Borrar (y la tecla Supr), Añadir compás, Quitar el último y Deshacer, según el diseño aprobado (depende de T021, T030 y T031)
- [ ] T034 [US2] Ejecutar `tests/features/editor.feature` y dejar en verde los escenarios de la historia 2

**Checkpoint**: La historia 2 funciona sola, sin servidor ni MIDI.

---

## Phase 4: User Story 3 - Escuchar la partitura (Priority: P1)

**Goal**: Reproducir con tempo fiable, tipo de compás, bucle, metrónomo y el kit predefinido en memoria.

**Independent Test**: Con el kit cargado, pulsar Play y comprobar sonido, indicador, tempo, bucle, metrónomo y cambio de tipo de compás.

### Tests for User Story 3 ⚠️

- [ ] T035 [P] [US3] Escribir `tests/features/transport.feature` con los escenarios de AC-02, AC-40 a AC-55 (incluidos AC-50 y AC-51), AC-108, AC-109, AC-114, AC-134 y AC-135
- [ ] T036 [P] [US3] Escribir los pasos en `tests/steps/transport.steps.ts`
- [ ] T037 [P] [US3] Escribir `tests/unit/scheduler.test.ts` (AC-85): cada evento se programa contra el reloj de audio a menos de 1 ms de su instante teórico, sin error acumulado y nunca en el pasado, a 30 y a 260 BPM
- [ ] T038 [P] [US3] Escribir `tests/unit/timesig.test.ts` (AC-108, AC-109): al cambiar el tipo de compás las notas que no caben pasan al compás siguiente, una nota pasa entera con su duración, un grupo de tresillo se mueve completo, y las que no entran en 128 compases se descartan avisando

### Implementation for User Story 3

- [ ] T039 [P] [US3] Implementar `src/core/timesig.ts` con el cambio de tipo de compás (RF-18.5, RF-18.6, RF-08.5), que cuenta como un solo cambio de Deshacer
- [ ] T040 [US3] Implementar `src/audio/scheduler.ts`: horizonte de 120 ms, instante de cada evento calculado como `inicio + posición`, nunca sumando intervalos; el cambio de tempo vale desde el siguiente compás con un nuevo punto de referencia (RF-18.7, research R-04) (depende de T015)
- [ ] T041 [P] [US3] Implementar `src/audio/metronome.ts`: pulsos según el tipo de compás con acento en el primer tiempo, ajustado al tempo, con volumen propio
- [ ] T042 [P] [US3] Implementar `src/audio/gain.ts` con las ganancias de AC-11: normal 1,0; acento 1,3; fantasma 0,4; flam con dos ataques (adorno con ganancia 0,5 a 30 ms antes del tiempo y principal con 1,0 en el tiempo)
- [ ] T043 [US3] Implementar `src/audio/kit.ts`: descargar `manifest.json` y los samples, decodificarlos a `AudioBuffer` y exponer los estados `cargando`, `listo` y `error` con Reintentar (RF-21.3, RF-21.5); después de cargado, la reproducción no hace ningún pedido a la red (RNF-15) (depende de T045 y T046)
- [ ] T044 [US3] Implementar `src/audio/transport.ts`: Play desde el compás 1, Detener cancela los eventos pendientes (AC-46), bucle, metrónomo activable en cualquier momento; el tipo de compás queda deshabilitado mientras suena (RF-18.8) (depende de T040, T041, T042, T043)
- [ ] T045 [P] [US3] Preparar el kit en `supabase/seed/kit/`: 13 samples (uno por pieza) con licencia CC0 o de compra con redistribución, `manifest.json` con `piece`, `file`, `sizeBytes`, `license` y `origin`, tamaño total de 3 MB o menos (RNF-15); 👤 requiere que una persona elija o compre los samples; conviene empezarla pronto, porque T043 depende de ella
- [ ] T046 [US3] Implementar la carga del kit al backend local en `tests/support/supabase.ts` y la política del bucket `kit` (lectura pública) en `supabase/migrations/0001_kit_bucket.sql`
- [ ] T047 [US3] 🎨 Implementar `src/ui/components/Transport.tsx` con Play/Detener, bucle, tempo (número entre 30 y 260), selectores de tipo de compás, metrónomo con su volumen, indicador de posición sobre la partitura y el estado del kit ("Cargando kit" y error con Reintentar) (depende de T021 y T044)
- [ ] T048 [US3] Ejecutar `tests/features/transport.feature` y dejar en verde los escenarios de la historia 3

**Checkpoint**: Las historias 2 y 3 funcionan juntas. Fin del MVP sin servidor de identificación.

---

## Phase 5: User Story 1 - Entrar con mi cuenta de Google (Priority: P1)

**Goal**: Exigir sesión con Google, registrar y reconocer al usuario, mantener la sesión cifrada y permitir una sola sesión por cuenta.

**Independent Test**: Abrir la aplicación sin sesión, identificarse, comprobar que la sesión sobrevive a cerrar y reabrir, que cerrar sesión devuelve a la identificación y que otra computadora desplaza a la primera.

### Tests for User Story 1 ⚠️

- [ ] T049 [P] [US1] Escribir `tests/features/auth.feature` con AC-74, AC-75, AC-77, AC-78, AC-80 a AC-82, AC-94 a AC-98, AC-131, AC-136 y AC-137; AC-76 queda en la historia 4 porque necesita la biblioteca y AC-79 (Linux sin almacén de secretos) queda en T115
- [ ] T050 [P] [US1] Escribir los pasos en `tests/steps/auth.steps.ts`

### Implementation for User Story 1

- [ ] T051 [US1] **Spike R-10**: comprobar en el servidor local que una función de base de datos con privilegios del definidor puede consultar `auth.sessions` desde una política RLS usando el `session_id` del token, medir su costo por pedido y anotar el resultado en `specs/001-drum-sync/research.md`; si no es viable, volver al spec antes de seguir
- [ ] T052 [US1] Crear `supabase/migrations/0002_profiles.sql`: tabla `profiles` (`id` igual al del usuario de Auth, `name`, `email`, `created_at`), disparador que crea el perfil la primera vez que entra una cuenta (RF-33) y RLS: el usuario lee su perfil y no modifica ni borra el de otro (AC-102)
- [ ] T053 [US1] Crear `supabase/migrations/0003_session_guard.sql`: función `is_session_active()` con privilegios del definidor que comprueba el `session_id` del token, para usarla en todas las políticas RLS (RF-36.1, depende de T051)
- [ ] T054 [US1] Configurar en `supabase/config.toml` y en la documentación el proveedor Google con PKCE, el token de acceso de 1 hora y el ajuste "una sola sesión por usuario"
- [ ] T055 [P] [US1] Implementar `electron/secureStore.ts`: sesión cifrada con `safeStorage` en su API asíncrona, `status()` que en Linux trata el almacén `basic_text` (consultado después de `ready`) como "no disponible", y nada guardado en texto plano (RNF-13, research R-09)
- [ ] T056 [US1] Implementar `electron/main.ts`: registro del protocolo `drumsync://` (segunda instancia en Windows y Linux, `open-url` en macOS), permiso MIDI concedido y los demás rechazados, y navegación restringida
- [ ] T057 [US1] Implementar `electron/preload.ts` con `secureStore`, `auth.openLogin` (solo `https` del proveedor), `auth.onCallback` y `env` según `contracts/ipc-preload.md`
- [ ] T058 [US1] Implementar `src/data/session.ts`: login en el navegador del sistema, retorno por `drumsync://`, renovación automática, persistencia por el puente seguro (en el navegador de desarrollo solo en memoria), mensajes de RF-35.2 y aviso de RF-36.1 ante el rechazo del servidor
- [ ] T059 [US1] Implementar el cierre de sesión en `src/data/session.ts` (RF-36.2): mostrar solo la pantalla de identificación y descartar de la memoria la partitura abierta, el historial de Deshacer, la biblioteca, el mapeo MIDI y los umbrales, y borrar el borrador local; el kit puede quedar cargado
- [ ] T060 [P] [US1] Implementar `tests/support/auth.ts`: usuarios de prueba creados en el servidor local que reemplazan el login con Google en los escenarios, y los pasos comunes para iniciar sesión con un usuario de prueba en `tests/steps/session.steps.ts`; además ayudantes para AC-95, AC-96 y AC-97 que alteran la firma de una credencial real, generan una vencida, fabrican una credencial de Google falsa (firma inválida, destinatario ajeno o vencida) y reutilizan una credencial de renovación ya usada, y los escenarios comprueban que el servidor rechaza todo y no se inicia ninguna sesión. Limitación aceptada: no se reproduce una credencial firmada por Google con destinatario ajeno
- [ ] T061 [US1] 🎨 Implementar `src/ui/screens/Identification.tsx` con el botón "Identificarse con Google" y los estados "Almacén de claves no disponible", los dos mensajes de RF-35.2 y el aviso "Tu sesión se cerró porque iniciaste en otra computadora" (depende de T021 y T058)
- [ ] T062 [US1] 🎨 Implementar en `src/ui/App.tsx` la barrera de acceso: sin sesión solo se ve la identificación (RF-32) y el kit se descarga después del login (depende de T021, T043 y T058)
- [ ] T063 [US1] Ejecutar `tests/features/auth.feature` y dejar en verde los escenarios de la historia 1 (los de Electron se ejecutan en el proyecto de Electron)

**Checkpoint**: La identificación funciona; la sesión es única y está cifrada.

---

## Phase 6: User Story 4 - Guardar y encontrar mis partituras (Priority: P1)

**Goal**: Autoguardar la partitura en curso, guardar con nombre en la biblioteca, abrir, eliminar y empezar una partitura nueva, con confirmaciones y errores definidos.

**Independent Test**: Escribir una partitura, cerrar y reabrir; guardarla como "Shuffle lento", abrirla, modificarla, volver a guardarla y eliminarla.

### Tests for User Story 4 ⚠️

- [ ] T064 [P] [US4] Escribir `tests/features/library.feature` con AC-56 a AC-64, AC-76, AC-81, AC-106, AC-110, AC-111, AC-116, AC-119, AC-123 y AC-130
- [ ] T065 [P] [US4] Escribir los pasos en `tests/steps/library.steps.ts`
- [ ] T066 [P] [US4] Escribir `tests/unit/library.test.ts`: nombre "no vacío, hasta 300 caracteres contados tal como se escribió", unicidad por usuario sin distinguir mayúsculas ni espacios en los extremos, y "cambios sin guardar" (difiere de su última versión guardada; si nunca se guardó, al menos una nota)

### Implementation for User Story 4

- [ ] T067 [US4] Crear `supabase/migrations/0004_scores.sql` según `contracts/database.md`: `scores` con `name` no vacío y de hasta 300 caracteres, `name_key` único por `(owner_id, name_key)`, `tempo` de 30 a 260, `measure_count` de 1 a 128, `format_version`, `data`, y `current_scores` con `revision` que el servidor incrementa en cada escritura; ambas con RLS (`owner_id = auth.uid()` y `is_session_active()`), sin permitir cambiar `owner_id`
- [ ] T068 [P] [US4] Implementar `src/core/library.ts`: `nameKey`, validación del nombre y unicidad por usuario
- [ ] T069 [P] [US4] Implementar `src/core/dirty.ts`: `hasUnsavedChanges(current, lastSaved)`
- [ ] T070 [US4] Implementar `src/data/scoresRepo.ts`: listar (nombre, tempo, compases, fecha), abrir (aplicando migraciones, T013), guardar (crear o reemplazar) y eliminar; errores "No se pudo guardar", "No se pudo abrir" y "No se pudo eliminar" con reintentar sin cambiar la biblioteca ni la partitura en curso (RF-25.2)
- [ ] T071 [US4] Implementar `src/data/currentScoreRepo.ts`: leer y escribir la partitura en curso con la `revision` esperada
- [ ] T072 [US4] Implementar `src/data/autosave.ts`: guardado con espera corta y límite máximo de 5 s después del último cambio (RNF-16), reintento cada 5 s ante fallo con conexión y aviso "No se están guardando los cambios" que desaparece al guardar (RF-22.5)
- [ ] T073 [US4] Implementar en `src/ui/state/scoreStore.ts` la acción Nueva partitura (partitura vacía de 1 compás) con confirmación si hay cambios sin guardar, y la misma confirmación al abrir otra de la biblioteca (RF-22.3, RF-22.4, RF-25.1)
- [ ] T074 [US4] 🎨 Implementar `src/ui/screens/Library.tsx` con el listado, abrir, eliminar con confirmación, el diálogo de guardado con nombre y sus errores, y los diálogos de confirmación (depende de T021 y T070)
- [ ] T075 [US4] 🎨 Recuperar la partitura en curso al reabrir la aplicación en `src/ui/App.tsx` (RF-22.2) (depende de T021 y T071)
- [ ] T076 [US4] Ejecutar `tests/features/library.feature` y dejar en verde los escenarios de la historia 4

**Checkpoint**: Las historias 1, 2, 3 y 4 (todas P1) funcionan juntas.

---

## Phase 7: User Story 5 - Escribir tocando con la batería MIDI (Priority: P2)

**Goal**: Conectar la batería, mapearla y grabar tomas cuantizadas con articulación según la intensidad.

**Independent Test**: Con golpes MIDI simulados de marca de tiempo conocida, grabar una toma de 1 compás y comprobar notas, cuantización, fusión, cuenta previa, fin de toma y articulaciones.

### Tests for User Story 5 ⚠️

- [ ] T077 [P] [US5] Escribir `tests/features/midi.feature` con AC-18 a AC-25, AC-90 y AC-115
- [ ] T078 [P] [US5] Escribir `tests/features/recording.feature` con AC-16, AC-26 a AC-39, AC-132 y AC-133
- [ ] T079 [P] [US5] Escribir los pasos en `tests/steps/midi.steps.ts` y `tests/steps/recording.steps.ts`, incluidos los pasos comunes para inyectar golpes MIDI simulados (usan la parte MIDI de la capacidad de pruebas, T119)
- [ ] T080 [P] [US5] Escribir `tests/unit/quantizer.test.ts` (AC-26, AC-31, AC-38, AC-39, RNF-06) y `tests/unit/articulation.test.ts` (AC-132): fantasma si la intensidad es menor o igual al umbral de fantasma, acento si es mayor o igual al de acento, normal en el resto, y el flam nunca por intensidad

### Implementation for User Story 5

- [ ] T081 [P] [US5] Implementar `src/midi/midi.ts`: mapeo General MIDI de percusión por defecto (notas 35 a 59) y reasignación de cualquier nota de 0 a 127 a una pieza (RF-11)
- [ ] T082 [P] [US5] Implementar `src/midi/access.ts`: pedir Web MIDI, listar entradas y distinguir los estados "Web MIDI no está disponible", "Permiso MIDI denegado" y "No se pudo abrir la entrada MIDI", y detectar la desconexión (RF-10)
- [ ] T083 [P] [US5] Implementar `src/midi/hits.ts`: un golpe es un mensaje de nota con intensidad mayor que cero; soltar el pad o la intensidad cero se ignoran (RF-17.3); la marca de tiempo es `event.timeStamp`
- [ ] T084 [P] [US5] Implementar `src/core/articulation.ts`: articulación por intensidad con umbrales por defecto 30 y 100, y validación "enteros de 1 a 127 con fantasma menor que acento" (si no se cumple, no se guardan)
- [ ] T085 [US5] Implementar `src/core/quantizer.ts`: cuantizar al paso más cercano de la grilla elegida (semicorchea 12, corchea de tresillo 16, corchea 24, negra 48) con error máximo de medio paso, conservar golpes simultáneos, fusionar los de la misma pieza en el mismo paso tomando la articulación de la mayor intensidad, duración por la separación con el golpe siguiente (el último dura hasta el final del compás) y reemplazar lo escrito en los compases de la toma
- [ ] T119 [US5] Agregar a `src/test-seam/index.ts` la parte de MIDI simulado según `contracts/test-seam.md`: `midi.setAvailability`, `midi.connectDevice`, `midi.disconnectDevice`, `midi.hit` (intensidad 64 por defecto) y `midi.release`; el golpe inyectado recorre el mismo camino que uno real (mapeo, cuantización, evaluación) y la parte MIDI también queda fuera de la compilación de producción (depende de T016, T081, T082, T083 y T085)
- [ ] T086 [US5] Crear `supabase/migrations/0005_user_settings.sql`: tabla `user_settings` con `owner_id` único, `midi_mapping`, `ghost_threshold` y `accent_threshold` (enteros de 1 a 127, fantasma menor que acento; por defecto 30 y 100) y RLS con `is_session_active()`
- [ ] T087 [US5] Implementar `src/data/settingsRepo.ts` para guardar y leer el mapeo y los umbrales de la cuenta (RF-12, RF-14.9)
- [ ] T088 [US5] Implementar `src/audio/take.ts`: toma de 1 a 128 compases que siempre empieza en el compás 1; arranque al primer golpe o con cuenta previa de un compás con metrónomo aunque esté desactivado (los golpes de la cuenta no se escriben); termina al completar los compases, con Detener o con Esc conservando lo escrito; tempo y tipo de compás bloqueados (RF-18.8); la desconexión de la batería termina la toma (RF-10.6); una sola entrada de Deshacer
- [ ] T089 [US5] 🎨 Implementar `src/ui/screens/MidiPanel.tsx` con Conectar batería, el listado, el último golpe (nota, canal e intensidad), el editor del mapeo, los avisos de motivo y "Batería desconectada" (depende de T021)
- [ ] T090 [US5] 🎨 Implementar `src/ui/screens/RecordingPanel.tsx` con cantidad de compases (1 a 128), grilla, modo de arranque, umbrales de articulación, Grabar y Detener, y los botones de grabar deshabilitados con su motivo si no hay MIDI o el kit no está cargado (depende de T021 y T088)
- [ ] T091 [US5] Ejecutar `tests/features/midi.feature` y `tests/features/recording.feature` y dejarlos en verde

**Checkpoint**: Se puede escribir tocando; el editor sigue funcionando sin MIDI (AC-90).

---

## Phase 8: User Story 6 - Practicar con feedback de timing (Priority: P2)

**Goal**: Modo Práctica con color, desvío, notas omitidas, bucle y volumen.

**Independent Test**: Con golpes simulados de desvío conocido sobre una partitura de bombo en el tiempo 1 y caja en el tiempo 3 a 120 BPM, verificar colores, desvíos, omitidas, bucle y volumen.

### Tests for User Story 6 ⚠️

- [ ] T092 [P] [US6] Escribir `tests/features/practice.feature` con AC-52, AC-65 a AC-73, AC-112, AC-113, AC-118, AC-120 y AC-121
- [ ] T093 [P] [US6] Escribir los pasos en `tests/steps/practice.steps.ts`
- [ ] T094 [P] [US6] Escribir `tests/unit/practice.test.ts` con la tabla de AC-66 (0, ±100, ±101, ±150, ±250 y ±251 ms), AC-67, AC-68, AC-69, AC-70 y AC-121

### Implementation for User Story 6

- [ ] T095 [US6] Implementar `src/core/practice.ts`: comparar cada golpe con la nota esperada más cercana en el tiempo sin importar la pieza (a igual distancia, la de la misma pieza), desvío con signo (positivo tarde, negativo adelantado), verde hasta 100 ms, amarillo hasta 250 ms, rojo por encima, por pieza distinta o sin nota esperada a menos de 250 ms, nota omitida a los 250 ms de su instante, y cada golpe evaluado por separado (RF-29)
- [ ] T096 [US6] Implementar `src/audio/practice.ts`: la partitura avanza en silencio (ganancia cero), volumen de la partitura independiente del metrónomo y regulable sin detener, cuenta previa opcional de un compás que suena solo al arrancar con bucle, borrado del feedback de la vuelta anterior al empezar cada vuelta, nota esperada destacada, y detención si la batería se desconecta; desvío medido contra el reloj de audio
- [ ] T097 [US6] 🎨 Implementar `src/ui/screens/Practice.tsx` con la nota esperada destacada, la pieza esperada, el color y el desvío en milisegundos con signo de cada golpe, el indicador de nota omitida, el volumen de la partitura, la cuenta previa, el bucle y el botón Limpiar práctica (depende de T021 y T096)
- [ ] T098 [US6] Mantener visible el feedback al terminar la práctica sin bucle hasta que se inicie otra práctica, se edite la partitura o se pulse Limpiar práctica (RF-30.5, RF-30.6) en `src/ui/state/practiceStore.ts`
- [ ] T099 [US6] Ejecutar `tests/features/practice.feature` y dejarlo en verde

**Checkpoint**: El Modo Práctica funciona con golpes simulados.

---

## Phase 9: User Story 7 - Avisos de conexión con el servidor (Priority: P3)

**Goal**: Modo sin conexión con borrador local cifrado, aviso fijo y reconexión con resolución de conflictos.

**Independent Test**: Abrir sin Internet con y sin sesión, cortar la conexión en plena práctica y reconectar con y sin conflicto.

### Tests for User Story 7 ⚠️

- [ ] T100 [P] [US7] Escribir `tests/features/connection.feature` con AC-83, AC-84, AC-105, AC-122, AC-124, AC-125, AC-126, AC-127 y AC-129
- [ ] T101 [P] [US7] Escribir los pasos en `tests/steps/connection.steps.ts`
- [ ] T102 [P] [US7] Escribir `tests/unit/sync.test.ts`: con `baseRevision` igual a la del servidor el borrador se sube sin preguntar y si difiere se pregunta al usuario

### Implementation for User Story 7

- [ ] T103 [US7] Implementar `src/data/connectivity.ts`: detectar la conexión con un pedido liviano al servidor y con los fallos de los pedidos reales, no solo con el estado de red del navegador
- [ ] T104 [US7] Implementar `src/data/draft.ts`: borrador cifrado por el puente seguro que se guarda como máximo 5 s después del último cambio mientras no hay conexión, con `baseRevision`; pertenece a un usuario y nunca se muestra a otro; se borra al subirse, al cerrar sesión o al descartarlo, y se descarta si inicia otro usuario (RF-39.6, RF-39.8)
- [ ] T105 [US7] Implementar `src/data/sync.ts`: al volver la conexión subir el borrador sin preguntar si la revisión del servidor es la misma y, si difiere, preguntar cuál conservar y descartar la otra (RF-39.7)
- [ ] T106 [US7] 🎨 Implementar `src/ui/components/ConnectionBanner.tsx` (aviso fijo que no tapa el editor ni la práctica), la pantalla "Sin conexión" para quien no tiene sesión, el modo sin conexión con la biblioteca deshabilitada ("Sin conexión") y Play, grabación y práctica deshabilitados con "Sin conexión: el kit no está cargado", y el diálogo de conflicto (depende de T021)
- [ ] T107 [US7] Comprobar en `src/audio/transport.ts` que ninguna reproducción hace pedidos a la red con el kit ya cargado y la conexión cortada (AC-105)
- [ ] T108 [US7] Ejecutar `tests/features/connection.feature` y dejarlo en verde

**Checkpoint**: Todas las historias del spec funcionan.

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Requisitos de calidad transversales y cierre

- [ ] T109 [P] Escribir `tests/features/access-control.feature` y sus pasos con el cliente de solicitudes de Playwright (AC-99, AC-100, AC-101, AC-102): un usuario B con credenciales válidas no lee, modifica, borra ni crea filas de A llamando directamente al servidor, y sin credenciales el servidor rechaza todo
- [ ] T110 [P] Escribir `tests/features/quality.feature` y sus pasos con AC-85, AC-88, AC-89, AC-92, AC-93, AC-103 y AC-138 (tiempos, grilla, formato, kit y migraciones)
- [ ] T111 [P] Escribir `tests/features/performance.feature` con AC-86, AC-87, AC-91 y AC-104 y sus pasos en `tests/steps/performance.steps.ts`, que llaman a los scripts de medición de `tests/perf/`: 100 muestras por medición, red a 10 Mbps cuando corresponde, y el escenario falla si el p95 de las 100 muestras supera el límite; se ejecuta con Playwright en el equipo de referencia (constitución, principio I)
- [ ] T112 Configurar los empaquetadores de Electron Forge para Windows, macOS y Linux en `forge.config.ts` y escribir `tests/features/installers.feature` con AC-107 y sus pasos en `tests/steps/installers.steps.ts`: instala el paquete generado y comprueba con Playwright que llega a la pantalla de identificación; se ejecuta en el proyecto de Playwright de cada sistema operativo (RNF-17)
- [ ] T113 Verificar que la compilación de producción no incluye `window.__drumsyncTest` y que su inyección no está disponible (AC-128) en `tests/features/quality.feature`
- [ ] T114 Revisión de seguridad OWASP Top 10 de todo el código con la escala OWASP Risk Rating: RLS en cada tabla y bucket, ausencia de `service_role` en la app, endurecimiento de Electron (aislamiento de contexto, permisos, navegación) y cifrado de sesión y borrador; registrar el resultado en `specs/001-drum-sync/security-review.md` y corregir críticas y altas
- [ ] T115 Escribir `tests/features/linux-keystore.feature` con AC-79 y sus pasos en `tests/steps/linux-keystore.steps.ts`: en una distribución de Linux sin almacén de secretos, al intentar identificarse se muestra "Almacén de claves no disponible" y no se inicia sesión (research R-09); se ejecuta en el proyecto de Playwright de Linux
- [ ] T116 Crear `docs/estado-rf.json` y `scripts/generar-estado.mjs` con todos los RF del `PRD.md` (incluidos los nuevos, como RF-08.5, RF-09.1 y RF-14.8), que necesita la skill `actualizar-estado-rf` al hacer un push
- [ ] T117 Ejecutar la validación manual con batería real descrita en `quickstart.md` y los objetivos de producto SC-002 y SC-003
- [ ] T118 Ejecutar `npm run verify` completo y confirmar la trazabilidad: todos los AC del `PRD.md` tienen su escenario etiquetado en `tests/features`

---

## Phase 11: Despliegue (al final)

**Purpose**: Dejar el backend de producción funcionando antes de distribuir la aplicación. Se hace después de Polish y no bloquea el desarrollo. Las tareas marcadas **👤** necesitan una persona (cuentas, claves o pago) y no las puedo hacer yo.

- [ ] T120 👤 Crear el cliente OAuth en Google Cloud Console y configurarlo como proveedor Google en el proyecto de Supabase de producción (PRD: dependencias)
- [ ] T121 👤 Crear el proyecto de Supabase de producción con plan pago y copias de seguridad diarias, y aplicar todas las migraciones de `supabase/migrations/` (PRD: riesgos, el plan gratuito solo sirve para desarrollo)
- [ ] T122 👤 Subir el kit y su `manifest.json` al bucket `kit` de producción, con la licencia (CC0 o compra con redistribución) y el origen de cada sample ya registrados, y comprobar que el total es de 3 MB o menos (RNF-15, AC-103)
- [ ] T123 Verificar en producción el ajuste "una sola sesión por usuario", la función `is_session_active()`, Row Level Security en cada tabla y que la app solo lleva la clave pública, repitiendo AC-99 a AC-102 y AC-131 contra el proyecto real (RNF-14, RF-36.1)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: sin dependencias.
- **Foundational (Phase 2)**: depende de Setup y bloquea las historias, incluida la aprobación del diseño visual (T021). Con T021 cerrada, las tareas 🎨 ya no tienen otra espera que sus dependencias normales.
- **Historias (Phase 3 a 9)**: dependen de Foundational. Se hacen en orden de entrega del plan.
- **Polish (Phase 10)**: depende de las historias que cubre cada tarea.
- **Despliegue (Phase 11)**: depende de Polish, en particular de T112 (instaladores), T114 (revisión de seguridad) y T045 (samples con licencia). No bloquea el desarrollo.

### User Story Dependencies

- **US2 (editor)**: solo Foundational. No necesita servidor ni MIDI.
- **US3 (sonido)**: solo Foundational y los samples (T045); usa el editor para tener una partitura que reproducir. El kit se carga después del login cuando exista US1.
- **US1 (identificación)**: Foundational; el spike T051 va antes de T053. Integra el kit con la barrera de acceso.
- **US4 (biblioteca)**: depende de US1 (usuario y `is_session_active()`) y de US2.
- **US5 (MIDI y tomas)**: depende de US2, US3 y, para guardar el mapeo y los umbrales, de US1.
- **US6 (práctica)**: depende de US3 y US5 (MIDI).
- **US7 (conexión)**: depende de US1 y US4 (autoguardado y revisiones).

### Within Each User Story

- Los escenarios y sus pasos primero, y deben fallar.
- Núcleo (`src/core`) antes de audio, MIDI y notación.
- Datos antes de interfaz; las tareas 🎨 esperan el diseño aprobado.
- Al final de cada historia se ejecuta su `.feature` hasta dejarlo en verde y se hace el paso de verificación de commit.

### Parallel Opportunities

- En Setup, T002, T004, T005, T006, T007 y T008.
- En Foundational, T010, T012, T013, T014, T015, T017, T018, T019 y T020.
- En cada historia, los escenarios, los pasos y las pruebas unitarias (marcados [P]) en paralelo.
- Con más de una persona: tras Foundational, US2 y la preparación de US1 (T051 a T055) pueden avanzar a la vez; US5 y US6 tras US3.

---

## Parallel Example: User Story 2

```bash
# Escenarios y pruebas unitarias de la historia 2 a la vez:
Task: "Escribir tests/features/editor.feature (T022)"
Task: "Escribir tests/steps/editor.steps.ts (T023)"
Task: "Escribir tests/unit/rests.test.ts (T024)"
Task: "Escribir tests/unit/measures.test.ts (T025)"

# Módulos de núcleo independientes a la vez:
Task: "Implementar src/core/rests.ts (T026)"
Task: "Implementar src/core/measures.ts (T027)"
Task: "Implementar src/notation/staff.ts (T029)"
```

---

## Implementation Strategy

### MVP primero (historias 2 y 3)

1. Terminar Setup y Foundational (sin olvidar T021, el diseño visual).
2. Terminar la historia 2 (editor con mouse) y validarla sola.
3. Terminar la historia 3 (escuchar) y validarla. **Parar y validar**: ya se puede escribir y escuchar una partitura sin servidor.

### Entrega incremental

1. Agregar US1 (identificación), con el spike T051 primero, y luego US4 (biblioteca): con las cuatro historias P1 el producto guarda y recupera el trabajo en la nube.
2. Agregar US5 (MIDI y tomas) y US6 (práctica).
3. Agregar US7 (conexión), cerrar con Polish y terminar con el Despliegue (Phase 11).

### Reglas de entrega

- Cada commit: `npm run verify` en verde, análisis OWASP hecho y mensaje en Conventional Commits.
- Cualquier cambio de formato de partitura trae su partitura de prueba y su migración (RNF-09.1).
- Toda tabla o bucket con datos de usuario trae RLS y sus escenarios de acceso (constitución y AGENTS.md).

---

## Notes

- [P] = archivos distintos y sin dependencias pendientes.
- La etiqueta [Story] vincula la tarea con una historia para la trazabilidad.
- Cada historia debe poder completarse y probarse sola, salvo las dependencias indicadas.
- Los escenarios deben fallar antes de implementar.
- Evitar tareas vagas, conflictos en un mismo archivo y dependencias entre historias que rompan su independencia.
