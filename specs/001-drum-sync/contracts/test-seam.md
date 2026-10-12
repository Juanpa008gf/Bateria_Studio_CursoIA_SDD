# Contrato: capacidad de pruebas (RNF-18)

Permite que los escenarios Playwright prueben grabación, Modo Práctica, tiempos y fallos de conexión sin batería, audio ni servidor reales. Solo existe en el entorno de pruebas: la compilación de producción no la incluye y no hay forma de activarla desde la interfaz (AC-128).

La capacidad se expone como `window.__drumsyncTest`, que solo existe si la compilación se hizo con la variable de entorno de pruebas.

## MIDI simulado

| Función | Efecto |
|---|---|
| `midi.setAvailability(estado)` | `ok`, `unavailable`, `denied` o `busy`; produce los avisos de RF-10.3 a RF-10.5 |
| `midi.connectDevice(nombre)` | agrega una entrada simulada (por ejemplo "Midiplus Pro 9") |
| `midi.disconnectDevice()` | simula la desconexión (RF-10.6) |
| `midi.hit({ note, channel, velocity, timestamp })` | inyecta un golpe; si no se indica `velocity` se usa 64 |
| `midi.release({ note, channel })` | inyecta el mensaje de soltar el pad |

El golpe inyectado recorre el mismo camino que uno real (mapeo, cuantización, evaluación).

## Reloj de audio

| Función | Efecto |
|---|---|
| `clock.freeze()` | detiene el avance del reloj de audio |
| `clock.advance(ms)` | avanza el reloj la cantidad indicada |
| `clock.scheduled()` | devuelve los eventos programados (notas y clicks) con su instante, su ganancia y el instante teórico |

Con esto se verifican los 4 clicks de cuenta previa (AC-34), la ganancia de las notas en práctica (AC-65, AC-71) y el error máximo de 1 ms en 10 minutos (AC-85).

## Red

| Función | Efecto |
|---|---|
| `network.set(estado)` | `online`, `offline` o `serverError` |
| `network.requests()` | lista los pedidos hechos al servidor, para comprobar que la reproducción no pide nada (AC-105) |

## Datos de prueba

El servidor local de pruebas se prepara desde `tests/support` con usuarios, partituras y el kit de prueba. El inicio de sesión con Google se reemplaza por usuarios de prueba creados directamente en el servidor local.

## Reglas

- Ninguna función de esta capacidad puede estar disponible para el usuario en producción.
- La compilación de producción se verifica sin `window.__drumsyncTest` (AC-128).
