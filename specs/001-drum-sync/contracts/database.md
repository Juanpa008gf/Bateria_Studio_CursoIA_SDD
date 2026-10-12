# Contrato: base de datos y almacenamiento (Supabase)

La app habla directamente con Supabase mediante `supabase-js`, sin servidor propio. Este contrato define lo que el cliente puede leer y escribir. La app solo lleva la clave pública (*anon key*); la clave `service_role` nunca se distribuye (RNF-14).

## Regla general de acceso (RNF-14)

Toda tabla con datos de usuario tiene Row Level Security activado. Las políticas limitan lectura, inserción, modificación y borrado a las filas cuyo `owner_id` es el usuario del token, y exigen que la sesión del token siga activa (RF-36.1). Sin token, solo con la clave pública, ningún pedido devuelve datos de usuario (AC-101). Una tabla o bucket nuevo con datos de usuario no se acepta sin su política y sus escenarios de acceso.

## Tablas

### `profiles`
| Columna | Tipo | Regla |
|---|---|---|
| `id` | uuid, clave | igual al identificador del usuario de Auth |
| `name` | texto | del proveedor |
| `email` | texto | del proveedor |
| `created_at` | fecha y hora | |

- La fila la crea el servidor al primer ingreso de una cuenta (RF-33); el cliente no inserta perfiles.
- Política: el usuario lee su perfil. No modifica ni borra el de otro (AC-102).

### `scores` (biblioteca)
| Columna | Tipo | Regla |
|---|---|---|
| `id` | uuid, clave | |
| `owner_id` | uuid | usuario dueño; no se puede cambiar |
| `name` | texto | no vacío, hasta 300 caracteres, tal como se escribió |
| `name_key` | texto | nombre sin espacios en los extremos y en minúsculas |
| `tempo` | entero | 30 a 260 |
| `measure_count` | entero | 1 a 128 |
| `format_version` | entero | versión del formato (hoy 1) |
| `data` | json | partitura (`score-format-v1.md`) |
| `created_at`, `updated_at` | fecha y hora | |

- Unicidad: `(owner_id, name_key)`.
- Operaciones del cliente: listar (nombre, tempo, compases, fecha), abrir una, guardar (crear o reemplazar), eliminar.
- Errores esperados: nombre vacío, de más de 300 caracteres o repetido se rechazan sin cambiar nada (RF-23).

### `current_scores` (partitura en curso)
| Columna | Tipo | Regla |
|---|---|---|
| `owner_id` | uuid, clave | una fila por usuario |
| `data` | json | partitura |
| `format_version` | entero | |
| `revision` | entero | el servidor lo incrementa en cada escritura |
| `updated_at` | fecha y hora | |

- Operaciones: leer, escribir (reemplazo completo) con la `revision` esperada. Si la `revision` enviada no coincide con la del servidor, la escritura se rechaza para que la app resuelva el conflicto (RF-39.7).

### `user_settings`
| Columna | Tipo | Regla |
|---|---|---|
| `owner_id` | uuid, clave | una fila por usuario |
| `midi_mapping` | json | nota MIDI 0–127 a pieza |
| `ghost_threshold` | entero | 1 a 127, menor que `accent_threshold`; por defecto 30 |
| `accent_threshold` | entero | 1 a 127; por defecto 100 |

- La restricción de rango y orden de los umbrales se aplica también en el servidor (RF-14.9).

## Almacenamiento

### Bucket `kit` (lectura pública)
- Contiene los samples del kit predefinido y `manifest.json` con, por cada sample: `piece`, `file`, `sizeBytes`, `license` y `origin`.
- Escritura solo desde la administración del proyecto, nunca desde la app. El tamaño total es de 3 MB o menos (RNF-15, AC-103).
- Es el único bucket sin datos de usuario.

## Identificación (Auth)

- Proveedor: Google, con flujo authorization code y PKCE (RNF-10).
- El servidor valida el token de Google (emisor, destinatario, firma y vencimiento) al hacer login; la app no lo valida (RNF-11).
- Token de acceso de 1 hora, renovación automática con credencial de renovación de un solo uso (RNF-12).
- Una sola sesión activa por cuenta: al iniciar en otra computadora, las demás sesiones dejan de ser válidas (RF-36.1).

## Errores que el cliente debe tratar

| Situación | Efecto en la app |
|---|---|
| Credencial alterada, vencida o ya usada | pantalla de identificación con el mensaje de RF-35.2 |
| Sesión desplazada por otra computadora | pantalla de identificación con el aviso de RF-36.1 |
| Servidor no responde, con conexión | "No se pudo guardar/abrir/eliminar" con reintentar (RF-25.2); autoguardado con reintento cada 5 s (RF-22.5) |
| Sin conexión | aviso fijo y modo borrador (RF-39, RF-40) |
| Conflicto de `revision` | diálogo para elegir entre borrador y servidor (RF-39.7) |
