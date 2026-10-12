# Data Model: Drum Sync

Entidades del spec con sus campos, reglas de validación y estados. Los nombres de tablas y columnas del servidor están en `contracts/database.md` y el formato de la partitura en `contracts/score-format-v1.md`.

## Constantes de la grilla (RNF-04)

La negra equivale a 48 unidades enteras. Duración de cada figura en unidades: redonda 192, blanca 96, negra 48, corchea 24, semicorchea 12; corchea de tresillo 16. Un compás de numerador N y denominador D dura `N × (192 / D)` unidades (4/4 = 192, 3/4 = 144, 6/8 = 144).

## Entidades

### Piece (pieza del kit)
13 valores fijos: bombo, caja, hi-hat cerrado, hi-hat abierto, hi-hat con pie, tom agudo, tom medio, tom de piso, crash, ride, campana del ride, china y splash. Voz: pies para bombo y hi-hat con pie; manos para el resto.

### Note (nota)
- `id`: identificador único dentro de la partitura.
- `piece`: Piece.
- `start`: posición dentro del compás, entero de 0 a (duración del compás − 1).
- `duration`: unidades; coincide con la figura elegida o con la separación al siguiente golpe en una toma.
- `figure`: redonda, blanca, negra, corchea o semicorchea.
- `tuplet`: opcional; `groupId` compartido por las tres notas de un tresillo.
- `articulation`: normal, acento, fantasma o flam.
- Reglas: la nota debe caber en el compás (`start + duration` no excede su duración); dos notas de la misma pieza no pueden estar en el mismo `start`; los silencios no se guardan, se derivan (RNF-05).

### Measure (compás)
- `notes`: lista de Note. Longitud implícita por el tipo de compás de la partitura.

### Score (partitura)
- `formatVersion`: entero; hoy 1 (RNF-09).
- `tempo`: entero de 30 a 260; un valor fuera de rango se ajusta al límite más cercano (RF-18.3).
- `timeSignature`: numerador de 1 a 16 y denominador 2, 4, 8 o 16.
- `measures`: de 1 a 128 compases; nunca vacío.
- Reglas: al cambiar el tipo de compás, las notas que no caben pasan enteras al compás siguiente (un grupo de tresillo se mueve completo) y las que no entran en 128 compases se descartan con aviso (RF-18.5, RF-18.6, RF-08.5).

### LibraryEntry (partitura de la biblioteca)
- `id`, `ownerId`, `name`, `tempo`, `measureCount`, `formatVersion`, `data` (Score), `createdAt`, `updatedAt`.
- Reglas: nombre no vacío, hasta 300 caracteres contados tal como se escribió; único por usuario sin distinguir mayúsculas ni espacios en los extremos (clave `name_key`); sin límite de cantidad (RF-23).

### CurrentScore (partitura en curso)
- `ownerId` (única por usuario), `data` (Score), `revision` (entero que el servidor incrementa en cada escritura), `updatedAt`.

### OfflineDraft (borrador sin conexión)
- `userId`, `data` (Score), `baseRevision` (revisión de la partitura en curso de la que partió), `savedAt`.
- Reglas: cifrado; pertenece a un usuario y nunca se muestra a otro; se borra al subirse, al cerrar sesión o al descartarlo (RF-39.6, RF-36.2).

### UserSettings (configuración de la cuenta)
- `ownerId` (única por usuario), `midiMapping`, `ghostThreshold`, `accentThreshold`.
- `midiMapping`: asignación de nota MIDI (0–127) a Piece; por defecto General MIDI de percusión (35–59).
- Reglas: `ghostThreshold` y `accentThreshold` enteros de 1 a 127 con fantasma menor que acento; por defecto 30 y 100; si no se cumple, no se guardan (RF-14.9).

### Profile (perfil)
- `id` (el del usuario de Auth), `name`, `email`, `createdAt`. Lo crea el servidor la primera vez que una cuenta entra (RF-33).

### Session (sesión)
- `accessToken`, `refreshToken`; solo existe cifrada en la computadora. Una sola sesión activa por cuenta (RF-36.1).

### Kit
- `manifest`: lista de samples con `piece`, `file`, `sizeBytes`, `license` (CC0 o compra con redistribución) y `origin`; tamaño total de 3 MB o menos (RNF-15).
- En memoria: un `AudioBuffer` por pieza; no se guarda en disco.

### Take (toma)
- `measures` (1 a 128), `grid` (semicorchea, corchea de tresillo, corchea o negra), `start` (primer golpe o cuenta previa), `hits`.

### Hit (golpe)
- `note` MIDI (0–127), `channel`, `velocity` (1–127), `timestamp`. Un golpe de intensidad cero y los mensajes de soltar no son golpes (RF-17.3).
- En práctica se evalúa con `PracticeResult`.

### PracticeResult
- `hitId`, `expectedNote`, `deviationMs` (con signo: positivo tarde, negativo adelantado), `color` (verde hasta 100 ms, amarillo hasta 250 ms, rojo por encima, por pieza distinta o sin nota esperada a menos de 250 ms).
- `omitted`: nota esperada sin ningún golpe a 250 ms o menos, una vez pasados 250 ms desde su instante.
- Cada golpe se evalúa por separado (RF-29.6). Con bucle, el resultado de la vuelta anterior se borra al empezar la siguiente (RF-30.7).

## Máquinas de estado

### Transporte
`detenido` → `reproduciendo` | `grabando` | `practicando` → `detenido`.
- Tipo de compás deshabilitado en los tres estados activos; tempo deshabilitado en `grabando` y `practicando` (RF-18.8).
- Desconexión de la batería: `grabando` pasa a `detenido` conservando los golpes; `practicando` pasa a `detenido` dejando el feedback (RF-10.6).

### Toma
`esperandoPrimerGolpe` | `cuentaPrevia` → `grabando` → `terminada`. Termina al completar los compases, con Detener o con Esc, conservando lo escrito (RF-16).

### Conexión
`enLinea` ⇄ `sinConexión`.
- Al abrir sin conexión y sin sesión guardada: bloqueo con "Sin conexión" (RF-39.1).
- Con sesión guardada: modo sin conexión con borrador (RF-39.3).
- Al volver: `baseRevision` igual a la del servidor, se sube; distinta, se pregunta al usuario (RF-39.7).

### Sesión
`sinSesión` → `activa` → `sinSesión` (por cierre de sesión, revocación, vencimiento, alteración de credencial o inicio en otra computadora). Cada salida tiene su mensaje (RF-35.2, RF-36.1).

### Kit
`cargando` → `listo` | `error` (con "Reintentar" que vuelve a `cargando`). Play, grabación y práctica solo se habilitan en `listo` (RF-21.3, RF-21.5).
