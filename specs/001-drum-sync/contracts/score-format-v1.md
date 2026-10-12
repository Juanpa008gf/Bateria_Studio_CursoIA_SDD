# Contrato: formato de partitura, versión 1

Es el contenido de la columna `data` de `scores` y `current_scores`, y del borrador sin conexión. Toda versión nueva debe abrir sin intervención las partituras de las versiones anteriores, con el 100 % de sus notas en la misma posición (RNF-09), y traer una partitura de prueba de cada versión previa con su escenario de migración (RNF-09.1).

## Estructura

```json
{
  "formatVersion": 1,
  "tempo": 120,
  "timeSignature": { "numerator": 4, "denominator": 4 },
  "measures": [
    {
      "notes": [
        {
          "id": "n1",
          "piece": "snare",
          "start": 48,
          "duration": 48,
          "figure": "quarter",
          "articulation": "normal"
        },
        {
          "id": "n2",
          "piece": "hihat_closed",
          "start": 96,
          "duration": 16,
          "figure": "eighth",
          "articulation": "accent",
          "tuplet": { "groupId": "t1", "size": 3, "inSpaceOf": 2 }
        }
      ]
    }
  ]
}
```

## Campos

| Campo | Regla |
|---|---|
| `formatVersion` | entero; esta versión es 1 |
| `tempo` | entero de 30 a 260 |
| `timeSignature` | numerador de 1 a 16; denominador 2, 4, 8 o 16 |
| `measures` | de 1 a 128 compases; cada uno con su lista `notes` |
| `notes[].id` | cadena única dentro de la partitura |
| `notes[].piece` | `kick`, `snare`, `hihat_closed`, `hihat_open`, `hihat_pedal`, `tom_high`, `tom_mid`, `tom_floor`, `crash`, `ride`, `ride_bell`, `china`, `splash` |
| `notes[].start` | posición en unidades de la grilla desde el inicio del compás (negra = 48) |
| `notes[].duration` | duración en unidades; la nota debe caber en el compás |
| `notes[].figure` | `whole`, `half`, `quarter`, `eighth`, `sixteenth` |
| `notes[].articulation` | `normal`, `accent`, `ghost` o `flam` |
| `notes[].tuplet` | opcional; las tres notas de un tresillo comparten `groupId` |

## Reglas de coherencia

- Dos notas de la misma pieza no pueden tener el mismo `start` en un compás.
- Los silencios no se guardan: se derivan al dibujar a partir de las notas (RNF-05).
- Un grupo de tresillo siempre tiene sus tres notas en el mismo compás.

## Versionado y migración

- Al abrir una partitura, el módulo `src/core/migrations` la lleva a la versión actual aplicando las migraciones en orden. El usuario no convierte nada.
- Hoy solo existe la versión 1: no hay nada que migrar y el módulo parte vacío.
- Cada migración nueva incluye su partitura de prueba y su escenario antes de publicarse.
