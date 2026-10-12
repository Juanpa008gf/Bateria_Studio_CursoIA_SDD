# Research: Drum Sync

Decisiones que resuelven los puntos abiertos del contexto técnico. El stack base viene de `AGENTS.md` y no se reabre: React + TypeScript sobre Vite, VexFlow, Electron (Forge solo para empaquetar), Supabase, Vitest y npm.

Las decisiones marcadas **Validar** dependen de un comportamiento que debe comprobarse con una prueba corta antes de construir sobre ellas. No pude verificarlas contra la documentación en esta etapa.

## R-01. Escenarios Gherkin con Playwright

- **Decision**: archivos `.feature` ejecutados por `playwright-bdd`, que traduce Gherkin a pruebas de Playwright. Un archivo por grupo de AC; cada escenario lleva la etiqueta de su AC (por ejemplo `@AC-66`).
- **Rationale**: la constitución exige Gherkin automatizado con Playwright. `playwright-bdd` mantiene un solo ejecutor y evita sumar un segundo (Cucumber) con su propia configuración.
- **Alternatives considered**: Cucumber.js con Playwright como librería (dos ejecutores, más configuración); pruebas Playwright sin `.feature` (no cumple Gherkin).

## R-02. Dos modos de ejecución para las pruebas

- **Decision**: Playwright corre cada escenario en dos proyectos: Chrome o Edge contra `localhost:5174` y Electron mediante su API de lanzamiento. Los escenarios que dependen de `safeStorage`, `drumsync://` o el instalador (AC-79, AC-94, AC-98, AC-107, AC-127) corren solo en Electron.
- **Rationale**: AGENTS.md pide que el cliente funcione en navegador para desarrollo, pero la distribución es Electron.
- **Alternatives considered**: solo navegador (no cubre `safeStorage` ni protocolo); solo Electron (más lento para el desarrollo diario).

## R-03. Capacidad de pruebas (RNF-18)

- **Decision**: módulo `src/test-seam` que expone en `window` una API de inyección de golpes MIDI, reloj de audio controlable y simulación de red, y que solo se incluye cuando la compilación se hace con la variable de entorno de pruebas. La compilación de producción no lo incluye. Contrato en `contracts/test-seam.md`.
- **Rationale**: Chromium en Playwright no tiene dispositivos MIDI ni audio real; la inyección es la única forma de probar AC-26, AC-66 y AC-85. AC-128 verifica que producción no lo tenga.
- **Alternatives considered**: puerto MIDI virtual del sistema (depende de cada sistema operativo); mocks solo en Vitest (no cubren el flujo completo).

## R-04. Reloj y programación de audio (RNF-01)

- **Decision**: cada nota y click se programa contra `AudioContext.currentTime` con un horizonte de 120 ms. El instante de cada evento se calcula como `inicioDeReproducción + posiciónEnSegundos`, nunca sumando intervalos al evento anterior. Un cambio de tempo (RF-18.7) crea un nuevo punto de referencia al inicio del siguiente compás.
- **Rationale**: elimina el error acumulado que exige RNF-01 y permite el cambio de tempo sin deriva.
- **Alternatives considered**: temporizadores de JavaScript (jitter de decenas de ms); reproducir con `setInterval` y sumar intervalos (acumula error).

## R-05. Kit de sonidos en memoria (RNF-15)

- **Decision**: el kit vive en un bucket público de Supabase Storage con un manifiesto (`manifest.json`) que lista cada sample con tamaño, origen y licencia. La app lo descarga tras el login, lo decodifica a `AudioBuffer` y solo entonces habilita Play, grabación y práctica. Si falla, muestra el error con "Reintentar" (RF-21.5). No se guarda copia local.
- **Rationale**: cumple RF-21 y AGENTS.md (la reproducción nunca pide nada a la red). El manifiesto hace verificable AC-103.
- **Alternatives considered**: empaquetar el kit dentro de la app (contradice RF-21.1, "servido desde el servidor"); caché local (fuera de alcance).

## R-06. Web MIDI en Electron

- **Decision**: usar Web MIDI desde el renderizador y conceder el permiso `midi` desde el proceso principal con un manejador de permisos. La marca de tiempo de cada golpe es `event.timeStamp`, y el desvío se mide contra el reloj de audio. El mapeo General MIDI (notas 35–59) vive en `src/midi/midi.ts`, versionado en el repo.
- **Rationale**: PRD, dependencias. Mantiene el aislamiento: sin permiso solo se deshabilitan grabación y práctica (RNF-07).
- **Alternatives considered**: módulo nativo de MIDI en el proceso principal (más código, rompe la ejecución en navegador).

## R-07. Notación con VexFlow

- **Decision**: render con VexFlow en un pentagrama de percusión con clave neutra, dos voces (manos con plicas arriba, pies con plicas abajo), barras por tiempo y tresillos con corchete y número. Los silencios se derivan al dibujar a partir de las notas; no se guardan. La detección de clic convierte (x, y) en (pieza, posición en la grilla) con la geometría que expone VexFlow.
- **Rationale**: RF-01 a RF-07 y RNF-05 (el compás siempre cierra exacto).
- **Alternatives considered**: render propio en canvas (mucho trabajo y sin beneficios frente a VexFlow).

## R-08. Autenticación con Google y protocolo `drumsync://`

- **Decision**: Supabase Auth con Google, flujo authorization code con PKCE. El login se abre en el navegador del sistema (`shell.openExternal`) y vuelve a la app por `drumsync://`, registrado en Electron con `setAsDefaultProtocolClient`. En Windows y Linux el retorno llega por el evento de segunda instancia y en macOS por `open-url`.
- **Rationale**: RNF-10 y la restricción de Google contra vistas web embebidas.
- **Alternatives considered**: ventana embebida (Google la bloquea); servidor local de retorno en un puerto (abre un puerto en la computadora).

## R-09. Sesión y borrador cifrados (RNF-13, RF-39.6)

- **Decision**: el proceso principal de Electron guarda la sesión y el borrador con `safeStorage.encryptString` en un archivo de la carpeta de datos de la app y los expone al cliente solo por el puente de preload (`contracts/ipc-preload.md`). Antes de habilitar el login se comprueba que el cifrado esté disponible; en Linux se comprueba además que el almacén elegido no sea el de texto plano. En el navegador de desarrollo la sesión vive solo en memoria y no se guarda borrador.
- **Rationale**: AC-79 y AC-98 exigen no guardar nada en texto plano ni en `localStorage` o IndexedDB.
- **Alternatives considered**: `localStorage` (prohibido); archivo con clave propia (la clave quedaría junto al archivo).
- **Verificado en la documentación de Electron**: `safeStorage.getSelectedStorageBackend()` existe desde Electron 25.5.0 (con retroportes a 26 y 27), es solo para Linux y devuelve `basic_text` cuando no hay almacén de secretos disponible; en ese caso los datos quedan prácticamente sin protección. Hay que llamarlo después del evento `ready`, porque antes devuelve `unknown`. La documentación recomienda además la API asíncrona (`encryptStringAsync` y `decryptStringAsync`) frente a la síncrona, que podría retirarse.
- **Decisión derivada**: elegir una versión de Electron igual o posterior a 25.5.0, usar la API asíncrona y tratar `basic_text` como "Almacén de claves no disponible" (RF-35.1).

## R-10. Una sola sesión activa por cuenta (RF-36.1)

- **Decision**: al iniciar sesión se invalidan las demás sesiones de la cuenta. Para que el servidor rechace el siguiente pedido de la sesión desplazada, aunque su token siga vigente, las políticas RLS comprueban además que la sesión del token siga activa mediante una función de base de datos con privilegios del definidor que consulta las sesiones de Auth. La app muestra la pantalla de identificación con el aviso del RF-36.1 al recibir el rechazo.
- **Rationale**: solo verificar firma y vencimiento (RNF-11) deja la sesión desplazada válida hasta 1 hora.
- **Alternatives considered**: acortar la vida del token (más renovaciones, no resuelve el caso); ajuste de "una sesión por usuario" del proveedor, que además depende del plan.
- **Verificado en la documentación de Supabase**: todo token de acceso lleva un `session_id` que corresponde a la clave de la tabla `auth.sessions`, y que no exista esa fila significa que el usuario cerró sesión. Además existe el ajuste "una sola sesión por usuario", que conserva la sesión más reciente y termina las demás, pero se aplica **al renovar el token**, por lo que el efecto se nota a intervalos del vencimiento del token (hoy 1 hora; Supabase no recomienda bajar de 5 minutos). Por eso el ajuste solo no cumple el "siguiente pedido es rechazado" de RF-36.1; hace falta la comprobación en el servidor del `session_id` en cada pedido.
- **Decisión confirmada**: se combinan el ajuste "una sola sesión por usuario" de Supabase (cierra las sesiones viejas al renovar) y la comprobación del `session_id` en las políticas RLS (rechazo inmediato en el siguiente pedido). Se acepta una consulta extra por pedido.
- **Pendiente (spike)**: la prueba de que una función de base de datos con privilegios del definidor puede consultar `auth.sessions` desde una política RLS, y la medición de su costo por pedido.

## R-11. Borrador sin conexión y sincronización (RF-39.x)

- **Decision**: el borrador guarda la partitura más `baseRevision`, el número de revisión de la partitura en curso del servidor de la que partió. Al reconectar, si `baseRevision` coincide con la revisión actual del servidor se sube sin preguntar; si no, se pregunta al usuario. La partitura en curso lleva un contador de revisión que el servidor incrementa en cada escritura. La conectividad se detecta con un pedido liviano al servidor y con los fallos de los pedidos reales, no solo con el estado de red del navegador.
- **Rationale**: define "la misma partitura" de RF-39.7 sin comparar contenido.
- **Alternatives considered**: comparar contenido completo (costoso e impreciso); marcas de tiempo (sensibles al reloj de cada computadora).

## R-12. Backend de pruebas

- **Decision**: la CLI de Supabase levanta un backend local (base, Auth y Storage) para las pruebas. Los escenarios de control de acceso (AC-99 a AC-102) llaman a la API directamente con el cliente de solicitudes de Playwright, sin pasar por la interfaz. El inicio de sesión con Google se reemplaza en pruebas por usuarios de prueba creados directamente en el servidor local.
- **Rationale**: pruebas repetibles sin tocar el proyecto real ni depender de Google.
- **Alternatives considered**: proyecto Supabase compartido (datos mezclados y pruebas lentas); mocks de `supabase-js` (no prueban RLS).

## R-13. Autoguardado

- **Decision**: guardado con espera corta después del último cambio y límite máximo de 5 s (RNF-16); ante un fallo con conexión, reintento cada 5 s y aviso "No se están guardando los cambios" (RF-22.5).
- **Rationale**: cumple RNF-16 con margen sin saturar el servidor.
- **Alternatives considered**: guardar en cada cambio (mucho tráfico); guardar solo al cerrar (pierde trabajo).

## R-14. Historial de Deshacer

- **Decision**: instantáneas inmutables de la partitura con un máximo de 100, que se pierden al cerrar la aplicación (RF-09.1). Una toma grabada o un cambio de tipo de compás cuenta como un solo cambio.
- **Rationale**: simple y comparte estructura entre instantáneas, lo que evita copias completas de 128 compases.
- **Alternatives considered**: patrón de comandos con inversa (más código por cada operación).

## R-15. Proceso de verificación antes de cada commit

- **Decision**: un script `npm run verify` ejecuta Vitest y todos los escenarios Playwright en ambos modos. El análisis OWASP (con la escala OWASP Risk Rating) se hace sobre el diff antes del commit, con ayuda de la skill `security-review`. Si algún escenario falla, se consulta a un humano.
- **Rationale**: constitución, principios II y III.
- **Alternatives considered**: gancho de git automático (los fallos requieren decisión humana, así que no puede ser solo automático).

## R-16. Versiones de las dependencias

- **Decision**: al iniciar el proyecto se instalan las versiones estables más recientes, se fijan exactas en `package.json` y se sube `package-lock.json` al repo. Las actualizaciones se hacen a propósito, con la suite completa en verde. Restricción conocida: Electron igual o posterior a 25.5.0 (R-09).
- **Rationale**: AGENTS.md dice que las versiones las fija `package.json`; fijarlas exactas evita que una actualización automática rompa el timing o las pruebas sin aviso. Las versiones concretas las resuelve `npm` al instalar.
- **Alternatives considered**: rangos compatibles (se actualizan solos y pueden cambiar el comportamiento entre dos instalaciones).

## Pendientes que pasan a tareas

- Spike de R-10: que una función de base de datos con privilegios del definidor pueda consultar `auth.sessions` desde una política RLS, y medir su costo por pedido.
- Prueba de R-09 en una distribución de Linux sin almacén de secretos, para confirmar el aviso "Almacén de claves no disponible".
- ~~Equipo de referencia para medir los p95~~ — **Resuelto**: se usa la computadora de desarrollo, que queda documentada en el repo. Características registradas el 2026-10-12: AMD Ryzen 7 Pro 7735U (8 núcleos, 16 hilos), 15 GB de RAM, Windows 11 Enterprise 10.0.26200 y Node v24.18.0. La red se limita a 10 Mbps cuando el requisito lo pide (AC-91, AC-104). Si cambia el equipo, se actualiza este registro y se vuelven a medir los p95. Limitación aceptada: los p95 medidos valen para este equipo, no garantizan el mismo resultado en otros (en particular Linux y macOS, RNF-17).
- Licencia de los samples del kit antes de publicar (AC-103).
