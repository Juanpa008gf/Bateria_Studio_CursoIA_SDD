# Contrato: puente entre Electron y el cliente

El cliente corre en el renderizador de Electron con aislamiento de contexto activado y sin acceso a Node. Todo lo que necesita del sistema operativo pasa por un objeto `window.drumsync` que expone `electron/preload.ts`. Cada función valida sus argumentos en el proceso principal.

En el navegador de desarrollo (Chrome o Edge en localhost) `window.drumsync` no existe: el cliente usa una versión en memoria que no persiste nada (RNF-13).

## Almacén seguro

| Función | Entrada | Resultado |
|---|---|---|
| `secureStore.status()` | — | `{ available: boolean, reason?: string }`. En Linux, `available` es falso si el sistema solo ofrece almacenamiento en texto plano |
| `secureStore.saveSession(session)` | credenciales de la sesión | confirma el guardado cifrado |
| `secureStore.loadSession()` | — | sesión o nada |
| `secureStore.clearSession()` | — | borra la sesión |
| `secureStore.saveDraft(userId, draft)` | borrador y `baseRevision` | confirma el guardado cifrado |
| `secureStore.loadDraft(userId)` | — | borrador del usuario o nada; nunca devuelve el de otro usuario |
| `secureStore.clearDraft()` | — | borra el borrador |

Reglas:
- Si `status().available` es falso, la app no permite iniciar sesión y muestra "Almacén de claves no disponible" (RF-35.1, AC-79).
- Nada se guarda en texto plano en la carpeta de datos, en `localStorage` ni en IndexedDB (RNF-13, AC-98).
- Si inicia sesión un usuario distinto del dueño del borrador, el borrador se descarta (RF-39.8).

## Identificación

| Función | Efecto |
|---|---|
| `auth.openLogin(url)` | abre `url` en el navegador del sistema. Solo acepta direcciones `https` del proveedor de identidad configurado |
| `auth.onCallback(callback)` | entrega la dirección `drumsync://` con la que el navegador vuelve a la app; el proceso principal la reenvía solo si su forma es válida |

El protocolo `drumsync://` se registra en la instalación. En Windows y Linux el retorno llega por la segunda instancia de la aplicación y en macOS por el evento de apertura de dirección.

## Entorno

| Función | Resultado |
|---|---|
| `env.platform()` | `windows`, `macos` o `linux` |
| `env.isElectron` | verdadero |

## Permisos

El proceso principal concede el permiso MIDI al renderizador y rechaza los demás permisos que la app no usa. Si se deniega o falla, el cliente muestra "Permiso MIDI denegado" o "No se pudo abrir la entrada MIDI" según el caso (RF-10.3 a RF-10.5).

## Prohibiciones de seguridad

- El renderizador no recibe `nodeIntegration` ni acceso al sistema de archivos.
- No se expone ninguna función genérica (ejecutar comandos, leer rutas arbitrarias).
- Solo se navega a direcciones permitidas; los enlaces externos se abren en el navegador del sistema.
