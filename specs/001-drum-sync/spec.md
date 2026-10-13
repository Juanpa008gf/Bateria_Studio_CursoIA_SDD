# Feature Specification: Drum Sync — Escribir, escuchar y practicar batería

**Feature Branch**: `001-drum-sync`

**Created**: 2026-10-12

**Status**: Draft

**Input**: User description: "Generá el spec a partir del PRD.md"

**Fuente**: `PRD.md` (PRD-001). Cada historia, requerimiento y criterio cita los IDs del PRD (RF, RNF, AC) para mantener la trazabilidad exigida por la constitución.

## Clarifications

### Session 2026-10-12

- Q: ¿Qué pasa con las notas ya escritas cuando se cambia el tipo de compás de una partitura que ya tiene notas? → A: Las notas que no caben en el nuevo compás pasan al compás siguiente, que se añade si hace falta.
- Q: ¿Qué pasa si el cambio de tipo de compás obliga a superar los 128 compases? → A: Se descartan las notas que no entran, avisando al usuario.
- Q: ¿Cómo empieza el usuario una partitura nueva y vacía? → A: Con un botón "Nueva partitura" que vacía el editor y pide confirmación si hay cambios sin guardar en la biblioteca.
- Q: ¿Qué ve el baterista cuando termina el Modo Práctica sin bucle? → A: El color y el desvío de cada golpe y las notas omitidas quedan visibles hasta que inicie otra práctica o edite la partitura, y un botón "Limpiar práctica" permite borrarlos.
- Q: ¿Tiene límite el nombre de una partitura y la cantidad de partituras por usuario? → A: El nombre admite hasta 300 caracteres y no hay límite de cantidad de partituras.
- Q: ¿El límite de 300 caracteres se cuenta antes o después de quitar los espacios de los extremos? → A: Antes: se cuentan tal como los escribió el usuario, incluidos los espacios de los extremos.
- Q: ¿Qué pasa si el kit de sonidos no se puede descargar o decodificar? → A: Se muestra un aviso de error con botón "Reintentar" y las funciones de sonido siguen deshabilitadas hasta que el kit cargue.
- Q: ¿Qué pasa si la batería se desconecta durante una toma o una práctica? → A: Aviso "Batería desconectada"; la toma termina conservando los golpes escritos y la práctica se detiene con el feedback visible.
- Q: ¿Qué pasa si falla un autoguardado teniendo conexión? → A: Se reintenta cada 5 s y se muestra un aviso visible mientras no se logre guardar.
- Q: ¿Cuántos cambios se pueden deshacer y se conserva el historial al cerrar la aplicación? → A: Hasta 100 cambios; el historial se pierde al cerrar la aplicación.
- Q: ¿El Modo Práctica permite una cuenta previa? → A: Sí, opcional, de un compás, marcada por el metrónomo aunque esté desactivado; los golpes durante la cuenta no reciben feedback.
- Q: ¿Debe el spec enumerar las pantallas y estados de la interfaz? → A: Sí, con la lista de pantallas y el contenido de cada una (controles y estados), sin detalles visuales (sección "Pantallas y Estados").
- Q: ¿Cuándo hay "cambios sin guardar en la biblioteca"? → A: Cuando la partitura en curso difiere de su última versión guardada en la biblioteca; si nunca se guardó, cuando tiene al menos una nota.
- Q: Al cambiar el tipo de compás, ¿qué pasa con una nota larga o un tresillo cortado por el final del nuevo compás? → A: La nota pasa entera al inicio del compás siguiente, conservando su duración; el grupo de tresillo se mueve completo.
- Q: ¿La unicidad del nombre se evalúa por usuario o entre todos los usuarios? → A: Solo dentro de la biblioteca de cada usuario; dos usuarios pueden usar el mismo nombre.
- Q: En Modo Práctica con bucle, ¿qué pasa con el feedback de la vuelta anterior? → A: Se borra al empezar cada vuelta; solo se ve la vuelta actual y, al terminar, la última.
- Q: Si dos golpes caen cerca de la misma nota esperada, ¿cómo se evalúan? → A: Cada golpe se evalúa por separado; ambos reciben su propio color y desvío.
- Q: El aviso de pérdida de conexión, ¿es fijo o temporal? → A: Fijo en un lugar visible, sin tapar el editor ni la práctica, mientras no haya conexión; desaparece al volver.
- Q: Con sesión guardada y sin conexión al abrir, ¿se permite entrar al editor? → A: Sí, en modo sin conexión con un borrador local cifrado de la partitura en curso, que se sube al volver la conexión; la biblioteca queda deshabilitada y el kit no se guarda localmente. Si al reconectar difiere de la versión del servidor, el usuario elige cuál conservar. Esto reemplaza la regla anterior y modifica AGENTS.md y RNF-13.
- Q: ¿La exclusión de accesibilidad y de otros idiomas es una decisión explícita? → A: Sí: quedan fuera de esta versión por decisión de alcance y se reevalúan en una versión posterior.
- Q: ¿Cómo se agregan las pruebas de migración cuando exista una versión posterior del formato? → A: Todo cambio de formato incluye una partitura de prueba de cada versión anterior publicada y su escenario de migración antes de publicarse.
- Q: ¿Qué se descarta al cerrar sesión? → A: Todo lo del usuario en pantalla y memoria (partitura abierta, historial de Deshacer, biblioteca, mapeo MIDI y umbrales) y el borrador local; el kit puede quedar cargado.
- Q: ¿La aplicación registra eventos de seguridad? → A: No: queda fuera de alcance de forma explícita; se usan los registros del proveedor de identidad y del servidor.
- Q: ¿Qué mensaje ve el usuario ante un fallo de sesión? → A: Dos mensajes: "Tu sesión venció, identifícate de nuevo" (vencida o revocada) y "No se pudo verificar tu sesión, identifícate de nuevo" (alterada o ya usada); la sesión desplazada por otra computadora mantiene su aviso propio.
- Q: ¿Qué pasa si se cambia tempo, tipo de compás o volumen con algo sonando? → A: El tempo puede cambiarse durante la reproducción y vale desde el siguiente compás; en toma y práctica el tempo queda bloqueado; el tipo de compás queda bloqueado en reproducción, toma y práctica; los volúmenes siguen ajustables.
- Q: ¿Qué articulación reciben las notas al grabar una toma? → A: La define la intensidad del golpe: fantasma hasta 30, acento desde 100 y normal en el resto, con umbrales ajustables por cuenta; el flam nunca se asigna por intensidad. Esto reemplaza "notas normales" de RF-14.2 y el recorte de "dinámica automática" del PRD.
- Q: ¿Qué pasa si se usa la misma cuenta en dos computadoras? → A: Solo se permite una sesión activa por cuenta: al iniciar sesión en otra computadora se cierra la anterior.
- Q: ¿Qué pasa si el servidor falla al guardar, abrir o eliminar una partitura con conexión? → A: Mensaje de error ("No se pudo guardar/abrir/eliminar") con opción de reintentar; la biblioteca y la partitura en curso no cambian.
- Q: ¿Cómo se borra una nota con el mouse? → A: Clic sobre una nota la selecciona y Supr (o un botón Borrar) la elimina; el borrado cuenta como un cambio de Deshacer y no pide confirmación.
- Q: ¿Qué pasa con lo escrito cuando la conexión se corta con la aplicación ya abierta y luego vuelve? → A: El mismo mecanismo que al abrir sin conexión: se guarda en el borrador local y, al reconectar, se sube sin preguntar si el servidor no cambió, o se pregunta al usuario si cambió.
- Q: ¿El spec exige poder simular golpes MIDI y controlar el reloj de audio en las pruebas? → A: Sí, como requisito (FR-048 / RNF-18), solo en el entorno de pruebas y no disponible en producción.
- Q: ¿SC-002 y SC-003 tienen prueba automática? → A: No: se marcan como validación manual con usuarios, igual que O-4 y O-1 del PRD.
- Q: ¿Unificar los términos del spec? → A: Sí: se agrega la sección "Glosario" y se usan "tipo de compás" (4/4) frente a "compás" (unidad), y "color" y "desvío" en lugar de "resultado".
- Q: ¿Abrir una partitura de la biblioteca debe pedir confirmación igual que "Nueva partitura"? → A: Sí, con la misma regla de cambios sin guardar (se unifica el comportamiento; ya no queda fuera de alcance).
- Q: ¿Grabar una toma y cambiar el tipo de compás deben tratar igual las notas que no entran en 128 compases? → A: Sí: toda operación que supere los 128 compases descarta las notas que no entran y muestra el mismo aviso. Hoy solo puede ocurrir con el cambio de tipo de compás.

## User Scenarios & Testing *(mandatory)*

Drum Sync es un editor de partitura de batería que suena y sirve para practicar con una batería electrónica MIDI. Cada baterista entra con su cuenta de Google y encuentra sus partituras desde cualquier computadora. Las historias están ordenadas por importancia; cada una entrega valor por sí sola.

### User Story 1 - Entrar con mi cuenta de Google (Priority: P1)

Un baterista abre la aplicación y se identifica con su cuenta de Google. La primera vez se registra; las siguientes entra directo. Solo ve sus propios datos y la sesión se mantiene entre cierres de la aplicación.

**Why this priority**: Todo el producto exige sesión iniciada (editor, biblioteca y práctica). Sin esto no se puede usar nada ni guardar nada en la cuenta.

**Independent Test**: Se puede probar abriendo la aplicación sin sesión, identificándose con una cuenta de Google y comprobando que se llega al editor, que la sesión sobrevive a cerrar y reabrir, y que cerrar sesión devuelve a la pantalla de identificación.

**Acceptance Scenarios**:

1. **Given** que no hay sesión iniciada, **When** se abre la aplicación, **Then** solo se muestra la pantalla para identificarse con Google y no se puede llegar al editor, la biblioteca ni el Modo Práctica. (AC-74)
2. **Given** una cuenta de Google que nunca entró a Drum Sync, **When** el usuario completa la identificación, **Then** se crea un único perfil con su nombre y su email y su biblioteca aparece vacía. (AC-75)
3. **Given** un usuario registrado con la partitura "Shuffle lento" en su biblioteca, **When** se identifica con la misma cuenta en otra computadora, **Then** entra a su mismo perfil, ve "Shuffle lento" y sigue habiendo un solo perfil. (AC-76)
4. **Given** un usuario con sesión iniciada, **When** cierra la aplicación y la reabre más de 1 hora después con conexión, **Then** entra directo al editor sin identificarse otra vez. (AC-77)
5. **Given** un usuario con sesión iniciada cuya sesión se revoca en el servidor, **When** la aplicación intenta renovarla, **Then** la sesión se cierra y se muestra la pantalla de identificación. (AC-78)
6. **Given** un usuario con sesión iniciada, **When** cierra sesión y reabre la aplicación, **Then** se muestra la pantalla de identificación. (AC-80)
7. **Given** un sistema sin almacén seguro para proteger la sesión, **When** se intenta identificarse, **Then** se muestra "Almacén de claves no disponible" y no se inicia sesión. (AC-79)
8. **Given** el usuario A con "Shuffle lento" guardada, **When** A cierra sesión y B entra en la misma computadora, **Then** "Shuffle lento" no aparece en el listado de B. (AC-82)
9. **Given** el usuario A con sesión iniciada, **When** guarda la partitura "Shuffle lento" en la biblioteca, **Then** la partitura guardada queda registrada a nombre de A. (AC-81)
10. **Given** un usuario con sesión iniciada en la computadora 1, **When** inicia sesión con la misma cuenta en la computadora 2, **Then** la computadora 2 entra al editor y la 1 muestra la identificación con "Tu sesión se cerró porque iniciaste en otra computadora", sin que la partitura en curso guardada por la 2 se modifique. (AC-131)
11. **Given** un usuario con sesión vencida o revocada que la aplicación no puede renovar, **When** se muestra la identificación, **Then** el mensaje es "Tu sesión venció, identifícate de nuevo"; si la credencial fue alterada o ya se usó, el mensaje es "No se pudo verificar tu sesión, identifícate de nuevo". (AC-136)
12. **Given** un usuario A con una partitura abierta y cambios en el historial de Deshacer, **When** cierra sesión y luego B inicia sesión en la misma computadora, **Then** B no ve la partitura abierta de A, Deshacer no tiene cambios y no aparecen la biblioteca, el mapeo MIDI ni los umbrales de A; el kit no se vuelve a descargar. (AC-137)

---

### User Story 2 - Escribir una partitura con el mouse (Priority: P1)

El baterista escribe un compás de batería haciendo clic en el pentagrama: elige la figura y la articulación, y la altura del clic decide la pieza del kit mientras la posición horizontal decide el momento rítmico. Los silencios, las barras y los grupos de tresillo se dibujan solos. Puede añadir y quitar compases y deshacer cambios.

**Why this priority**: Es el núcleo del producto y no depende de MIDI. Con esta historia ya se puede capturar una idea.

**Independent Test**: Escribir un compás de 4/4 con bombo, caja y hi-hat, comprobar posiciones, silencios, barras y voces, añadir y quitar compases y deshacer.

**Acceptance Scenarios**:

1. **Given** un compás con negra de bombo en el tiempo 1 y negra de caja en el tiempo 3, **When** se muestra, **Then** se ve un pentagrama de percusión con clave neutra, bombo en el 1.er espacio y caja en el 3.er espacio. (AC-01)
2. **Given** un compás vacío, **When** se escribe una negra de cada pieza del kit, **Then** cada una se dibuja en su altura y con su cabeza convencionales (tabla de AC-03). (AC-03)
3. **Given** la figura Negra elegida, **When** se hace clic a la altura de la caja en el tiempo 2 del compás 1, **Then** aparece una negra de caja en el tiempo 2. (AC-04)
4. **Given** la figura Corchea con tresillo activado, **When** se escriben tres golpes de hi-hat en el tiempo 1, **Then** los tres entran en ese tiempo, unidos por una barra y con el número 3. (AC-06)
5. **Given** un compás de 4/4 con una sola negra de caja en el tiempo 1, **When** se muestra, **Then** la voz de manos tiene silencio de negra en el tiempo 2 y de blanca en los tiempos 3–4, y al agregar una negra de caja en el tiempo 4 los silencios se recalculan solos. (AC-07)
6. **Given** un compás sin notas, **When** se muestra, **Then** tiene un único silencio de compás completo. (AC-08)
7. **Given** ocho corcheas de hi-hat cerrado y bombo en los tiempos 1 y 3, **When** se muestra, **Then** el hi-hat lleva plicas hacia arriba, el bombo hacia abajo y la voz de pies muestra silencios de negra en los tiempos 2 y 4. (AC-09)
8. **Given** ocho corcheas de hi-hat, **When** se muestra, **Then** se ven cuatro grupos de dos unidos por barra, sin barras entre tiempos distintos. (AC-10)
9. **Given** un golpe de caja, **When** se aplica cada articulación (normal, acento, fantasma, flam), **Then** cada una se dibuja y suena de forma distinta según la tabla de AC-11. (AC-11)
10. **Given** una partitura de 2 compases, **When** se pulsa Añadir compás, **Then** tiene 3 compases y los dos primeros no cambian; con 3 compases con notas, Quitar el último deja 2 con las mismas notas. (AC-12, AC-13)
11. **Given** una partitura de 1 compás, **When** se intenta quitar el último, **Then** sigue teniendo 1 compás con sus notas; y con 128 compases no se puede añadir otro. (AC-14, AC-15)
12. **Given** una nota recién escrita, **When** se pulsa Deshacer, **Then** la partitura vuelve a como estaba antes. (AC-17)
13. **Given** una partitura a la que se le hicieron 101 cambios, **When** se pulsa Deshacer 101 veces, **Then** solo se deshacen los últimos 100; y al cerrar y reabrir la aplicación no queda ningún cambio para deshacer. (AC-117)
14. **Given** un compás de 4/4 vacío, **When** se escribe cada figura en el tiempo 1, **Then** ocupa en la grilla 192 unidades la redonda, 96 la blanca, 48 la negra, 24 la corchea y 12 la semicorchea. (AC-05)
15. **Given** un compás con una negra de caja en el tiempo 1, **When** se hace clic sobre esa nota, **Then** queda seleccionada y la partitura no cambia; un clic en un lugar sin nota con la figura Negra elegida escribe una nota nueva. (AC-139)
16. **Given** un compás de 4/4 con una negra de caja en el tiempo 1 y una de bombo en el tiempo 3, **When** se selecciona la caja y se pulsa Supr (o el botón Borrar), **Then** la caja desaparece, el bombo no cambia y la voz de manos muestra un silencio de compás completo; Deshacer la restaura. (AC-140)

---

### User Story 3 - Escuchar la partitura (Priority: P1)

El baterista reproduce la partitura con tempo fiable, tipo de compás elegido, bucle y metrónomo opcional. Los sonidos son de un kit predefinido de samples reales, distinto para cada pieza, que el usuario no tiene que cargar.

**Why this priority**: "Lo que se escribe suena, y suena a batería" es un objetivo P0 del PRD (O-2). Es la segunda pieza de la cadena de valor.

**Independent Test**: Escribir una partitura con una nota de cada pieza, esperar a que cargue el kit, pulsar Play y comprobar sonido, indicador de posición, tempo, bucle y metrónomo.

**Acceptance Scenarios**:

1. **Given** un usuario que acaba de entrar, **When** el kit todavía se descarga, **Then** Play, la grabación y el Modo Práctica están deshabilitados y se muestra "Cargando kit"; al terminar se habilitan y el texto desaparece. (AC-55)
2. **Given** una partitura con una nota de cada pieza, **When** se reproduce, **Then** cada pieza suena con su propio sample, distinto de los demás. (AC-54)
3. **Given** una partitura de 2 compases en 4/4 a 60 BPM, **When** se pulsa Play y suena el tiempo 3 del compás 1, **Then** el indicador de posición está sobre el tiempo 3 del compás 1. (AC-02)
4. **Given** una reproducción detenida en el compás 2, **When** se pulsa Play, **Then** vuelve al tiempo 1 del compás 1 y la primera nota suena en el primer instante. (AC-45)
5. **Given** una reproducción en curso, **When** se pulsa Detener, **Then** se cancelan los eventos de audio pendientes, el indicador se detiene y el botón vuelve a Play. (AC-46)
6. **Given** un compás con el bucle activado, **When** termina, **Then** vuelve al compás 1 sin detenerse. (AC-47)
7. **Given** el tempo en 100 BPM, **When** se escribe 128, **Then** pasa a 128; con 20 pasa a 30 y con 300 pasa a 260. (AC-40, AC-41, AC-42)
8. **Given** el selector de tipo de compás, **When** se despliega, **Then** el numerador ofrece 1 a 16 y el denominador 2, 4, 8 y 16; al elegir 3/4 con el metrónomo activo hay 3 clicks por compás. (AC-43, AC-44)
9. **Given** una partitura en 4/4 a 120 BPM con metrónomo activado, **When** se pulsa Play, **Then** se programan 4 clicks por compás, el primero acentuado; si se desactiva el metrónomo no se programa ningún click más. (AC-49, AC-48)
10. **Given** una partitura de 1 compás en 4/4 con notas en los tiempos 1 y 4, **When** se cambia el tipo de compás a 3/4, **Then** la partitura tiene 2 compases y la nota del tiempo 4 pasa al compás 2, sin perder ninguna; una blanca del tiempo 3 pasa entera al compás 2 con su duración, y un grupo de tresillo se mueve completo. (AC-108)
11. **Given** una partitura de 128 compases con notas en todos, **When** se cambia a un tipo de compás con menos pulsos, **Then** se descartan las notas que no entran, se avisa al usuario y Deshacer restaura la partitura anterior. (AC-109)
12. **Given** el volumen del metrónomo en cero, **When** suena la partitura, **Then** los clicks no se oyen y las notas no cambian de volumen. (AC-53)
13. **Given** un usuario que acaba de entrar y cuyo kit falla al descargarse, **When** termina el intento, **Then** se muestra un aviso de error con el botón Reintentar y Play, la grabación y el Modo Práctica siguen deshabilitados; al reintentar con éxito el aviso desaparece y se habilitan. (AC-114)
14. **Given** el metrónomo activado, **When** se graba una toma y cuando se practica, **Then** en ambos casos se programan sus clicks. (AC-50)
15. **Given** el metrónomo desactivado, **When** se reproduce, se graba una toma sin cuenta previa y se practica, **Then** en ningún caso se programa un click. (AC-51)
16. **Given** una partitura de 4 compases a 100 BPM en reproducción con metrónomo, **When** se cambia el tempo a 140 BPM durante el compás 2, **Then** el compás 2 termina a 100 BPM y desde el compás 3 las notas y los clicks van a 140 BPM, sin error de 1 ms o más. (AC-134)
17. **Given** una reproducción, una toma o una práctica en curso, **When** se miran los controles de tempo y de tipo de compás, **Then** el tipo de compás está deshabilitado en los tres casos y el tempo en la toma y la práctica (no en la reproducción); el volumen del metrónomo y el de la partitura en práctica siguen ajustables, y al detener se habilitan. (AC-135)

---

### User Story 4 - Guardar y encontrar mis partituras (Priority: P1)

El trabajo se autoguarda en la cuenta del usuario y al reabrir la aplicación vuelve tal cual estaba. Además, el usuario guarda partituras con nombre en su biblioteca, las lista, las abre y las elimina con confirmación.

**Why this priority**: O-5 del PRD: nada de lo escrito se pierde y todo está en un solo lugar.

**Independent Test**: Escribir una partitura, cerrar y reabrir; guardarla como "Shuffle lento", abrirla, modificarla, volver a guardarla y eliminarla.

**Acceptance Scenarios**:

1. **Given** una partitura nunca guardada en la biblioteca, **When** se cierra y reabre la aplicación, **Then** aparece la misma partitura con su tempo y sus notas. (AC-56)
2. **Given** una partitura de 2 compases a 120 BPM, **When** se guarda como "Shuffle lento", **Then** el listado muestra ese nombre, 120 BPM, 2 compases y la fecha de hoy. (AC-57)
3. **Given** "Shuffle lento" en la biblioteca, **When** se abre, **Then** el editor la carga con el mismo tempo, compases y notas. (AC-58)
   - **Given** una partitura en curso con cambios sin guardar en la biblioteca, **When** se abre otra de la biblioteca, **Then** se pide confirmación; al cancelar no cambia nada y al confirmar se abre la elegida. (AC-123)
4. **Given** "Shuffle lento" abierta y modificada, **When** se vuelve a guardar, **Then** la fecha se actualiza y sigue habiendo una sola entrada. (AC-59)
5. **Given** el usuario A con "Shuffle lento" en su biblioteca, **When** el usuario B guarda una partitura con ese mismo nombre en la suya, **Then** se guarda sin error y la de A no cambia. (AC-119)
6. **Given** una partitura, **When** se intenta guardar con nombre vacío, de más de 300 caracteres o con el nombre de otra partitura (sin distinguir mayúsculas ni espacios en los extremos), **Then** no se guarda y la biblioteca no cambia. (AC-60, AC-61, AC-111)
7. **Given** una partitura en la biblioteca, **When** se pide eliminarla, **Then** se pide confirmación; al cancelar sigue en el listado y al confirmar desaparece. (AC-62, AC-63, AC-64)
8. **Given** una partitura en curso con cambios sin guardar en la biblioteca, **When** se pulsa Nueva partitura, **Then** se pide confirmación; al cancelar la partitura no cambia y al confirmar el editor queda con una partitura vacía de 1 compás. Si no hay cambios sin guardar, o la partitura nunca guardada está vacía, se vacía sin pedir confirmación; una partitura nunca guardada con al menos una nota, o una abierta de la biblioteca y modificada, sí pide confirmación. (AC-110)
9. **Given** un usuario con conexión, **When** escribe una nota, espera 6 s y la aplicación se cierra de golpe, **Then** al reabrir la nota está en la partitura en curso. (AC-106)
10. **Given** un usuario con conexión cuyo servidor rechaza los autoguardados, **When** escribe una nota, **Then** el sistema reintenta cada 5 s y muestra "No se están guardando los cambios"; cuando el guardado se completa, la nota queda guardada y el aviso desaparece. (AC-116)
11. **Given** un usuario con conexión cuyo servidor falla, **When** guarda una partitura, abre una de la biblioteca o elimina una, **Then** se muestra "No se pudo guardar", "No se pudo abrir" o "No se pudo eliminar" con la opción de reintentar y no cambian ni la biblioteca ni la partitura en curso; al reintentar con el servidor disponible, la acción se completa. (AC-130)

---

### User Story 5 - Escribir tocando con la batería MIDI (Priority: P2)

El baterista conecta su batería electrónica, comprueba que está bien mapeada y graba una toma: sus golpes se escriben como notas normales, cuantizadas a la grilla elegida, y quedan editables con el mouse.

**Why this priority**: Es el diferencial del producto (capturar una idea tocándola), pero requiere hardware y el editor funciona sin él.

**Independent Test**: Con golpes MIDI simulados de marcas de tiempo conocidas, grabar una toma de 1 compás y comprobar notas, cuantización, fusión, cuenta previa y fin de toma.

**Acceptance Scenarios**:

1. **Given** una batería conectada por USB, **When** se pulsa Conectar batería, **Then** aparece en el listado de entradas y, al elegirla y golpear un pad, se muestran nota, canal e intensidad del último golpe. (AC-18, AC-19)
2. **Given** el mapeo por defecto, **When** se golpean pads que envían las notas 36 y 38, **Then** cuentan como bombo y caja; y al reasignar cualquier nota (0–127) a una pieza los golpes pasan a contar como esa pieza. (AC-22, AC-20, AC-21)
3. **Given** que el usuario reasignó una nota en una computadora, **When** entra con la misma cuenta en otra, **Then** la reasignación sigue ahí. (AC-23)
4. **Given** que la función MIDI no está disponible o su permiso se denegó, **When** se abre el editor, **Then** la grabación y el Modo Práctica están deshabilitados con el motivo "Web MIDI no está disponible" o "Permiso MIDI denegado", y escribir, guardar y reproducir funcionan igual. (AC-24, AC-90)
5. **Given** una entrada MIDI tomada por otra aplicación, **When** se la elige, **Then** se muestra "No se pudo abrir la entrada MIDI" y la grabación y el Modo Práctica siguen deshabilitados. (AC-25)
6. **Given** una toma de 1 compás a 120 BPM con grilla de semicorchea, **When** llegan un golpe de bombo a 0 ms y uno de caja a 510 ms, **Then** el bombo queda en el tiempo 1 y la caja en el tiempo 2, como notas normales editables. (AC-26)
7. **Given** esa toma, **When** llegan mensajes de soltar el pad o un golpe de intensidad cero, **Then** no se escribe nada. (AC-27)
8. **Given** compases con notas previas, **When** se graba sobre ellos, **Then** quedan solo los golpes grabados; si la toma es más larga que la partitura se añaden los compases que faltan. (AC-28, AC-29)
9. **Given** una toma con arranque al primer golpe, **When** pasan 3 s sin golpes, **Then** no empieza y no se escribe nada; al golpear la caja ese golpe queda en el tiempo 1. (AC-32)
10. **Given** una toma con cuenta previa y metrónomo desactivado, **When** arranca, **Then** se programan 4 clicks de cuenta (el primero acentuado) y los golpes durante la cuenta no se escriben. (AC-33, AC-34)
11. **Given** una toma en curso con golpes escritos, **When** se completa el compás, se pulsa Detener o se pulsa Esc, **Then** la toma termina y los golpes ya escritos se conservan. (AC-35, AC-36, AC-37)
12. **Given** golpes simultáneos y golpes repetidos de la misma pieza en el mismo paso, **When** se graban, **Then** los simultáneos se conservan como notas simultáneas y los repetidos se fusionan en una. (AC-38, AC-39)
13. **Given** una toma terminada, **When** se pulsa Deshacer, **Then** vuelve la partitura anterior a la toma. (AC-16)
14. **Given** una toma o una práctica en curso con golpes ya registrados, **When** se desconecta la batería, **Then** se muestra "Batería desconectada"; la toma termina conservando los golpes escritos y la práctica se detiene dejando visible el feedback. (AC-115)
15. **Given** el selector de cantidad de compases de la toma, **When** se despliega, **Then** ofrece como máximo 128 compases. (AC-30)
16. **Given** una toma de 1 compás en 4/4 a 120 BPM con grilla en negra, **When** llegan un golpe de bombo en el tiempo 1 y uno de caja en el tiempo 3, y el pad del bombo se suelta 100 ms después de golpearlo, **Then** el bombo se escribe como blanca y la caja, por ser el último golpe, como blanca hasta el final del compás. (AC-31)
17. **Given** una toma con los umbrales por defecto (fantasma 30, acento 100), **When** llegan golpes de caja de intensidad 30, 31, 99 y 100, **Then** se escriben como fantasma, normal, normal y acento, y ninguna como flam; dos golpes de caja de intensidad 20 y 110 en el mismo paso se fusionan en una nota con acento. (AC-132)
18. **Given** un usuario que ajusta los umbrales a fantasma 20 y acento 120, **When** hace login con la misma cuenta en otra computadora, **Then** siguen en 20 y 120; si intenta guardar fantasma 100 y acento 90, o un valor fuera de 1 a 127, no se guardan y los umbrales no cambian. (AC-133)

---

### User Story 6 - Practicar con feedback de timing (Priority: P2)

El baterista activa el Modo Práctica: la partitura avanza (en silencio o de fondo bajito) y él toca sobre ella. Cada golpe recibe color (verde, amarillo, rojo), la pieza esperada y el desvío en milisegundos con signo; las notas no tocadas se marcan como omitidas.

**Why this priority**: Responde la pregunta "¿estoy tocando esto bien?". Es el tercer pilar del producto y su función crítica, pero depende de la identificación, el kit y MIDI.

**Independent Test**: Con golpes simulados de desvío conocido sobre una partitura de bombo en el tiempo 1 y caja en el tiempo 3 a 120 BPM, verificar colores, desvíos, omitidas, bucle y volumen.

**Acceptance Scenarios**:

1. **Given** una partitura con notas en los tiempos 1 y 3, **When** se activa el Modo Práctica con metrónomo y se pulsa Play, **Then** las notas de la partitura no se oyen, el metrónomo suena y, pasada la nota del tiempo 1, la nota destacada es la del tiempo 3. (AC-65)
2. **Given** bombo esperado en el tiempo 1, **When** se toca con desvío de 0, ±100, ±101, ±150, ±250 y ±251 ms, **Then** el color es verde hasta 100 ms, amarillo hasta 250 ms y rojo por encima, y siempre se muestran la pieza esperada y el desvío con signo (positivo tarde, negativo adelantado). (AC-66)
3. **Given** crash esperado en el tiempo 1, **When** se toca bombo con 0 ms de desvío, **Then** el golpe se ve rojo por ser pieza distinta. (AC-67)
4. **Given** un golpe a igual distancia de dos notas esperadas, **When** se compara, **Then** se elige la de la misma pieza que el golpe. (AC-68)
5. **Given** un golpe sin ninguna nota esperada a menos de 250 ms, **When** se toca, **Then** se ve rojo. (AC-69)
6. **Given** que solo se toca el bombo, **When** pasan 250 ms desde el instante de la caja, **Then** la caja se marca como omitida con un indicador distinto de verde, amarillo y rojo. (AC-70)
7. **Given** una práctica con volumen de la partitura en 0 %, **When** se sube al 50 % sin detener, **Then** las notas empiezan a oírse, el metrónomo no cambia, la práctica no se interrumpe y el color y el desvío de cada golpe son iguales que con el volumen en 0 %. (AC-71, AC-72, AC-73)
8. **Given** una práctica sin bucle que terminó con golpes verdes, amarillos, rojos y notas omitidas, **When** se mira la partitura, **Then** el feedback sigue visible; **When** se pulsa Limpiar práctica, se edita la partitura o se inicia otra práctica, **Then** el feedback anterior desaparece. (AC-112, AC-113)
9. **Given** una partitura de 1 compás en práctica a 100 BPM con bucle y metrónomo, **When** termina el compás, **Then** vuelve a empezar, el metrónomo sigue a 100 BPM y los golpes siguen recibiendo feedback. (AC-52)
10. **Given** una partitura de 1 compás en 4/4 a 120 BPM en Modo Práctica con cuenta previa y el metrónomo desactivado, **When** arranca la práctica, **Then** se programan 4 clicks de cuenta, el primero acentuado, y los golpes durante la cuenta no reciben feedback; con bucle, la cuenta suena solo al arrancar. (AC-118)
11. **Given** una partitura de 1 compás en Modo Práctica con bucle y golpes con feedback en la primera vuelta, **When** empieza la segunda vuelta, **Then** desaparece el feedback de la primera y solo se ve el de la segunda; al detener la práctica queda visible el de la última vuelta. (AC-120)
12. **Given** bombo esperado en el tiempo 1, **When** se tocan dos golpes de bombo, uno a 0 ms y otro a 40 ms del tiempo 1, **Then** ambos se ven verdes, cada uno con su propio desvío. (AC-121)

---

### User Story 7 - Avisos de conexión con el servidor (Priority: P3)

La aplicación necesita conexión para identificarse, descargar el kit y usar la biblioteca. Si falta al abrirla sin sesión guardada, avisa y no da acceso; con sesión guardada abre el editor en modo sin conexión, con un borrador local cifrado que se sube al volver la conexión. Si la conexión se pierde durante el uso, avisa sin interrumpir lo que suena.

**Why this priority**: Es un caso de borde del uso normal; no entrega valor por sí solo pero evita que el usuario pierda trabajo sin saberlo.

**Independent Test**: Abrir la aplicación sin Internet y, aparte, cortar la conexión durante una práctica.

**Acceptance Scenarios**:

1. **Given** una computadora sin conexión y sin sesión guardada, **When** se abre la aplicación, **Then** se muestra "Sin conexión" y no se puede llegar al editor. (AC-83)
2. **Given** un usuario practicando, **When** se corta la conexión, **Then** aparece un aviso de falta de conexión, la reproducción no se detiene, ningún sonido se retrasa y el feedback sigue apareciendo. (AC-84)
3. **Given** el kit ya cargado y la conexión cortada, **When** se reproduce una partitura, **Then** no se hace ningún pedido de red y suena con todos los samples. (AC-105)
4. **Given** un usuario con el editor abierto, **When** se corta la conexión y pasan 60 s, **Then** el aviso sigue visible sin tapar el editor y desaparece al volver la conexión. (AC-122)
5. **Given** un usuario con sesión guardada y sin conexión, **When** abre la aplicación, **Then** el editor se abre con el aviso fijo, se puede escribir con el mouse, guardar en la biblioteca, abrir y eliminar quedan deshabilitados con el motivo "Sin conexión", y Play, la grabación y el Modo Práctica quedan deshabilitados con "Sin conexión: el kit no está cargado". (AC-124)
6. **Given** un usuario sin conexión que escribe una nota y espera 6 s, **When** cierra y reabre la aplicación sin conexión, **Then** la nota está en el borrador; al volver la conexión, si la partitura en curso del servidor es la misma de la que partió el borrador, se sube sin preguntar y el borrador local se borra. (AC-125)
7. **Given** un borrador local de una partitura que se editó desde otra computadora, **When** vuelve la conexión, **Then** se pregunta al usuario cuál conservar y la elegida queda como partitura en curso. (AC-126)
8. **Given** un borrador local sin subir, **When** el usuario cierra sesión, o inicia sesión otro usuario en la misma computadora, **Then** el borrador se borra o se descarta y nunca se muestra a otro; y no aparece en texto plano. (AC-127)
9. **Given** un usuario con el editor abierto y la partitura en curso ya guardada en el servidor, **When** se corta la conexión, escribe una nota y espera 6 s, **Then** la nota queda en el borrador local; al volver la conexión, si la partitura del servidor no cambió, la nota se sube sin preguntar, el borrador se borra y el aviso desaparece. (AC-129)

---

---

### Escenarios de calidad (requisitos no funcionales)

Escenarios de aceptación de los requisitos de tiempos, seguridad, compatibilidad y rendimiento (FR-037 a FR-047). Son transversales a las historias anteriores.

**Tiempos y precisión**

1. **Given** una reproducción simulada de 10 minutos, una a 30 BPM y otra a 260 BPM, **When** se registra el instante programado de cada nota y cada click, **Then** ninguno se aparta 1 ms o más de su instante teórico y ninguno se programa en el pasado respecto del reloj de audio. (AC-85, FR-037)
2. **Given** 100 golpes MIDI simulados durante una toma y otros 100 en Modo Práctica, **When** se mide el tiempo desde la marca de tiempo de cada golpe hasta que se dibuja en la partitura, **Then** el p95 es menor que 250 ms en ambos casos. (AC-86, FR-038)
3. **Given** 100 golpes MIDI simulados en Modo Práctica con desvíos conocidos, **When** se mide el feedback, **Then** el p95 del tiempo hasta el feedback visual es menor que 100 ms y cada desvío mostrado difiere del conocido en 20 ms o menos. (AC-87, FR-038)
4. **Given** la grilla rítmica, **When** se consulta cuántas unidades tiene la negra, **Then** tiene 48; la semicorchea ocupa 12, la corchea de tresillo 16 y la corchea 24; y al guardar y reabrir una partitura con esas figuras cada nota queda en la misma posición. (AC-88, FR-039)
5. **Given** un compás con tresillos junto a semicorcheas y un compás escrito por una toma grabada, **When** se calculan los silencios, **Then** en cada voz la suma de las figuras y los silencios es exactamente la duración del compás. (AC-89, FR-039)

**Rendimiento y carga**

6. **Given** un usuario con partitura en curso de 128 compases y conexión de 10 Mbps, **When** hace login 100 veces, **Then** el p95 del tiempo hasta ver la partitura en curso es menor que 3 s. (AC-91, FR-046)
7. **Given** el kit predefinido publicado, **When** se suman los tamaños de sus archivos, **Then** el total es de 3 MB o menos y cada sample tiene registrada una licencia CC0 o de compra con redistribución. (AC-103, FR-045)
8. **Given** un usuario con sesión iniciada y conexión de 10 Mbps, **When** hace login 100 veces, **Then** el p95 del tiempo hasta tener el kit decodificado en memoria es de 5 s o menos. (AC-104, FR-045)

**Compatibilidad de formato**

9. **Given** un conjunto de partituras de prueba, una por cada versión publicada del formato y con notas en todas las figuras y piezas del kit, **When** cada una se abre con la versión actual, **Then** se abre sin que el usuario convierta nada y con el 100 % de sus notas en la misma posición musical. (AC-92, FR-041)
10. **Given** una partitura guardada con la versión actual, **When** se lee la fila guardada en el servidor, **Then** lleva el número de versión de formato 1. (AC-93, FR-041)
    - **Given** un cambio de formato que define la versión N+1, **When** se prepara su publicación, **Then** existe una partitura de prueba de cada versión anterior publicada con su escenario de migración; sin eso, el cambio no se publica. (AC-138, FR-041)

**Identificación y sesión**

11. **Given** un usuario sin sesión, **When** pulsa identificarse con Google, **Then** el login se abre en una ventana externa a la aplicación y, al terminar, vuelve a ella. (AC-94, FR-042)
12. **Given** una credencial de acceso alterada, o vencida y sin credencial de renovación válida, **When** la aplicación la usa en un pedido, **Then** el servidor lo rechaza y se muestra la pantalla de identificación. (AC-95, FR-042)
13. **Given** una credencial de Google alterada, con destinatario ajeno a la aplicación o vencida, **When** se intenta hacer login con ella, **Then** el servidor la rechaza y no se inicia ninguna sesión. (AC-96, FR-042)
14. **Given** una credencial de renovación que ya se usó una vez, **When** se intenta usarla de nuevo, **Then** el servidor la rechaza y se muestra la pantalla de identificación. (AC-97, FR-042)
15. **Given** un usuario con sesión iniciada, **When** se inspeccionan la carpeta de datos de la aplicación y el almacenamiento del navegador, **Then** ninguna credencial de la sesión aparece en texto plano. (AC-98, FR-043)

**Control de acceso en el servidor**

16. **Given** un usuario B con credenciales válidas y los identificadores de una partitura de la biblioteca, de la partitura en curso y del mapeo MIDI del usuario A, **When** B las pide, las modifica o las borra llamando directamente al servidor sin pasar por la aplicación, **Then** cada pedido es rechazado, no se devuelve ningún dato de A y no se modifica ni borra nada. (AC-99, FR-044)
17. **Given** un usuario B con credenciales válidas, **When** intenta crear una fila indicando a A como dueño, **Then** el servidor la rechaza y no se crea ninguna fila. (AC-100, FR-044)
18. **Given** los pedidos de los dos escenarios anteriores, **When** se repiten sin credenciales de usuario, solo con la clave pública, **Then** el servidor los rechaza y no devuelve ningún dato de usuario. (AC-101, FR-044)
19. **Given** un usuario B con credenciales válidas y el identificador del perfil (nombre y email) del usuario A, **When** B lo pide, lo modifica o lo borra llamando directamente al servidor, **Then** el pedido es rechazado, no se devuelve ningún dato de A y no se modifica ni borra nada. (AC-102, FR-044)

**Plataformas**

20. **Given** el instalador de la aplicación en Windows, en macOS y en Linux, **When** se instala y se abre en cada uno, **Then** en los tres se llega a la pantalla de identificación. (AC-107, FR-047)

**Capacidad de pruebas**

21. **Given** el entorno de pruebas, **When** se inyecta un golpe MIDI simulado con marca de tiempo conocida y se adelanta el reloj de audio, **Then** el golpe se registra y se evalúa como uno real; en la versión de producción esa capacidad no está disponible. (AC-128, FR-048)

### Edge Cases

- Mapeo MIDI con notas extremas (0 y 127) y notas sin asignar: toda nota recibida de 0 a 127 es reasignable.
- Un golpe cae entre dos pasos de la grilla: se cuantiza al paso más cercano con error máximo de medio paso.
- Tresillos junto a semicorcheas y compases escritos por una toma: en cada voz las figuras y los silencios deben completar el compás exactamente.
- Toma que excede los 128 compases de la partitura: se añaden compases solo hasta el tope de 128.
- Último golpe de una toma: dura hasta el final del compás en que cae; la duración de cada nota se deduce de la separación con el golpe siguiente y nunca de cuándo se soltó el pad.
- Abrir una partitura de la biblioteca con cambios sin guardar en la partitura en curso: se pide confirmación antes de reemplazarla (misma regla que Nueva partitura).
- Quitar un compás con notas no pide confirmación.
- Sin conexión, los cambios se guardan en un borrador local cifrado (máximo 5 s después del último cambio) y se suben al volver la conexión; si el usuario cierra sesión sin haberlos subido, el borrador se pierde.
- Conflicto al reconectar: si la partitura en curso del servidor difiere de aquella de la que partió el borrador, el usuario elige cuál conservar.
- Con la sesión revocada o vencida al reconectar, se muestra la identificación y el borrador se conserva cifrado para ese usuario hasta que vuelva a identificarse; si entra otro usuario, se descarta.
- Dos partituras con el mismo nombre en distinto formato de mayúsculas o con espacios en los extremos cuentan como el mismo nombre.
- Cambiar el compás (por ejemplo de 4/4 a 3/4) con notas en tiempos que dejan de existir: esas notas pasan al compás siguiente, que se añade si hace falta, sin perder ninguna. Si con eso se superan los 128 compases, se descartan las notas que no entran y se avisa al usuario.
- Uso de la misma cuenta en dos computadoras: solo una sesión puede estar activa; la computadora que tenía la sesión anterior pasa a la identificación y, si tenía un borrador sin subir, se aplica la regla de reconexión del borrador (FR-036b).
- Partituras guardadas con versiones anteriores del formato deben abrirse sin intervención y sin perder notas.

## Requirements *(mandatory)*

### Functional Requirements

**Notación**
- **FR-001**: El sistema MUST mostrar la partitura en un pentagrama de percusión con clave neutra e indicar la posición actual durante la reproducción. (RF-01)
- **FR-002**: El sistema MUST permitir escribir las 13 piezas del kit (bombo, caja, hi-hat cerrado, abierto y con pie, tres toms, crash, ride, campana del ride, china y splash), cada una en su altura convencional. (RF-02)
- **FR-003**: El sistema MUST permitir escribir redonda, blanca, negra, corchea y semicorchea, y figuras en tresillo. (RF-03)
- **FR-004**: El sistema MUST calcular y mostrar los silencios que completan cada compás, separando manos (plicas hacia arriba, todas las piezas salvo bombo y hi-hat con pie) y pies (plicas hacia abajo, bombo y hi-hat con pie). (RF-04)
- **FR-005**: El sistema MUST unir con barras las corcheas y figuras menores dentro de un mismo tiempo, y marcar los tresillos con corchete y número. (RF-05)
- **FR-006**: El sistema MUST permitir marcar y dibujar de forma distinta las articulaciones normal, acento, nota fantasma y flam, y reproducir cada una con un sonido distinto. (RF-06)
- **FR-007**: El sistema MUST elegir la pieza por la altura del clic y la posición rítmica por la posición horizontal del clic. Un clic sobre una nota ya escrita MUST seleccionarla sin borrarla ni escribir otra, y un clic sobre un lugar sin nota MUST escribir una. (RF-07, RF-07.3)
- **FR-007a**: El sistema MUST permitir borrar la nota seleccionada con la tecla Supr o con un botón Borrar; el borrado cuenta como un cambio de Deshacer y los silencios del compás se recalculan solos. (RF-07.4)
- **FR-008**: El sistema MUST permitir añadir compases hasta un máximo de 128 y quitar el último, sin permitir dejar la partitura sin compases. Toda operación que obligue a superar los 128 compases MUST descartar las notas que no entran y mostrar el mismo aviso de notas descartadas (hoy solo ocurre con el cambio de tipo de compás, FR-020a, porque una toma arranca en el compás 1 y dura como máximo 128). (RF-08, RF-08.5)
- **FR-009**: El sistema MUST permitir deshacer cualquier cambio de la partitura, venga del mouse o de una toma grabada. El historial MUST conservar hasta los últimos 100 cambios y perderse al cerrar la aplicación. (RF-09, RF-09.1)

**Conexión con la batería MIDI**
- **FR-010**: El sistema MUST conectarse a una batería MIDI por USB y listar las entradas disponibles. (RF-10.1, RF-10.2)
- **FR-011**: Si la función MIDI no está disponible, el permiso se deniega o la entrada está tomada por otra aplicación, el sistema MUST deshabilitar la grabación y el Modo Práctica y mostrar el motivo correspondiente. Si la batería se desconecta durante una toma o una práctica, MUST mostrar "Batería desconectada", terminar la toma conservando los golpes ya escritos y detener la práctica dejando visible el feedback. (RF-10.3, RF-10.4, RF-10.5, RF-10.6)
- **FR-012**: El sistema MUST traer un mapeo por defecto de percusión General MIDI (notas 35–59), permitir reasignar cualquier nota 0–127 a una pieza del kit y guardar un único mapeo por cuenta. (RF-11, RF-12)
- **FR-013**: El sistema MUST mostrar en vivo el último golpe recibido (nota, canal e intensidad). (RF-13)

**Grabación de tomas**
- **FR-014**: El sistema MUST permitir grabar una toma de 1 a 128 compases, que siempre empieza en el compás 1, y escribir cada golpe como nota editable. (RF-14.1, RF-14.2, RF-14.6)
- **FR-014a**: El sistema MUST asignar a cada nota grabada una articulación según la intensidad del golpe (1 a 127): fantasma si es menor o igual que el umbral de fantasma, acento si es mayor o igual que el umbral de acento y normal en el resto; por defecto 30 y 100. El flam nunca se asigna por intensidad. Si dos golpes de la misma pieza se fusionan, la nota toma la articulación de la mayor de las dos intensidades. (RF-14.8)
- **FR-014b**: El sistema MUST permitir al usuario ajustar los dos umbrales (enteros de 1 a 127, con el de fantasma menor que el de acento; si no, no se guardan) y guardarlos en su cuenta, un par por cuenta, con el mismo control de acceso que el resto de sus datos. (RF-14.9)
- **FR-015**: El sistema MUST cuantizar los golpes a la grilla elegida (semicorchea, corchea de tresillo, corchea o negra) con error máximo de medio paso. (RF-14.3)
- **FR-016**: El sistema MUST reemplazar lo escrito en los compases de la toma, añadiendo los compases que falten hasta 128, y deducir la duración de cada nota de la separación con el golpe siguiente. (RF-14.4, RF-14.5, RF-14.7)
- **FR-017**: El sistema MUST permitir arrancar la toma al primer golpe o con una cuenta previa de un compás marcada por el metrónomo (aunque esté desactivado), sin escribir los golpes de la cuenta. (RF-15)
- **FR-018**: El sistema MUST terminar la toma al completar los compases, al pulsar Detener o al pulsar Esc, conservando los golpes ya escritos. (RF-16)
- **FR-019**: El sistema MUST conservar los golpes simultáneos, fusionar los de la misma pieza en el mismo paso e ignorar los mensajes que no son un golpe (soltar pad, intensidad cero). (RF-17)

**Transporte y sonido**
- **FR-020**: El sistema MUST permitir fijar el tempo entre 30 y 260 BPM (ajustando al límite más cercano si se escribe uno fuera de rango) y elegir el tipo de compás con numerador de 1 a 16 y denominador 2, 4, 8 o 16. (RF-18)
- **FR-020b**: Si se cambia el tempo durante una reproducción, el cambio MUST valer desde el siguiente compás, para las notas y el metrónomo. Mientras se graba o se practica el tempo MUST estar deshabilitado, y el tipo de compás MUST estar deshabilitado mientras se reproduce, se graba o se practica; se pueden cambiar al detener. El volumen del metrónomo y el de la partitura en práctica siguen ajustables. (RF-18.7, RF-18.8)
- **FR-020a**: Al cambiar el tipo de compás de una partitura con notas, el sistema MUST conservar todas las notas: las que no caben en el nuevo tipo de compás pasan al compás siguiente, que se añade si hace falta, y ninguna nota se descarta. Una nota cuya duración excede lo que queda del compás pasa entera al inicio del compás siguiente, conservando su duración, y un grupo de tresillo se mueve completo. La única excepción son las que no entren en el tope de 128 compases (FR-008): esas se descartan y el sistema MUST avisar al usuario de que se descartaron notas. El cambio se puede deshacer (FR-009). (RF-18.5, RF-18.6)
- **FR-021**: El sistema MUST reproducir la partitura desde el compás 1 al pulsar Play, detenerla con Detener y permitir reproducir en bucle. (RF-19)
- **FR-022**: El sistema MUST ofrecer un metrónomo activable en cualquier momento, que suene en la reproducción, la grabación y el Modo Práctica, marque el pulso con acento en el primer tiempo, siga el tempo y el compás de la partitura y tenga volumen propio. (RF-20)
- **FR-023**: El sistema MUST traer un kit predefinido de samples reales, igual para todos, con un sonido distinto por pieza, y no habilitar Play, la grabación ni el Modo Práctica hasta que esté cargado, mostrando que está cargando. Si el kit no se puede descargar o decodificar, MUST mostrar un aviso de error con un botón "Reintentar" y mantener deshabilitadas esas funciones hasta que el kit esté cargado. (RF-21, RF-21.5)

**Guardado y biblioteca**
- **FR-024**: El sistema MUST autoguardar la partitura en curso en la cuenta del usuario sin intervención manual y recuperarla al reabrir la aplicación. Si un autoguardado falla teniendo conexión, MUST reintentarlo cada 5 s y mostrar un aviso visible mientras no logre guardar. (RF-22, RF-22.5)
- **FR-024a**: El sistema MUST ofrecer una acción "Nueva partitura" que deje el editor con una partitura vacía de 1 compás. Si la partitura en curso tiene cambios que no están guardados en la biblioteca, MUST pedir confirmación antes de vaciarla. Hay cambios sin guardar cuando la partitura en curso difiere de su última versión guardada en la biblioteca; si nunca se guardó, cuando tiene al menos una nota. (RF-22.3, RF-22.4)
- **FR-025**: El sistema MUST permitir guardar en la biblioteca con un nombre, volver a guardar sobre una existente, y rechazar nombres vacíos, de más de 300 caracteres (contados tal como los escribió el usuario, incluidos los espacios de los extremos) o repetidos dentro de la biblioteca del mismo usuario (sin distinguir mayúsculas ni espacios en los extremos); dos usuarios distintos pueden usar el mismo nombre. La cantidad de partituras por usuario no tiene límite. (RF-23, RF-23.5)
- **FR-026**: El sistema MUST listar la biblioteca con nombre, tempo, cantidad de compases y fecha de última modificación, permitir abrir cualquier partitura, pidiendo confirmación antes de reemplazar la partitura en curso si tiene cambios sin guardar (misma regla que FR-024a), y eliminarla de forma definitiva solo tras confirmación. Si el servidor falla al guardar, abrir o eliminar con conexión, MUST mostrar "No se pudo guardar", "No se pudo abrir" o "No se pudo eliminar" con la opción de reintentar, sin cambiar la biblioteca ni la partitura en curso. (RF-24, RF-25, RF-25.1, RF-25.2, RF-26)

**Modo Práctica**
- **FR-027**: El sistema MUST ofrecer un Modo Práctica que haga avanzar la partitura, arranque con su sonido en silencio, destaque la próxima nota esperada y permita usar el bucle. MUST permitir arrancar con una cuenta previa opcional de un compás marcada por el metrónomo (aunque esté desactivado), sin evaluar los golpes durante la cuenta; con bucle, la cuenta suena solo al arrancar. (RF-27, RF-27.3, RF-28, RF-30.4)
- **FR-028**: El sistema MUST comparar cada golpe con la nota esperada más cercana en el tiempo (a igual distancia, la de la misma pieza) y calcular el desvío, positivo si es tarde y negativo si es adelantado. Cada golpe MUST evaluarse por separado: si dos golpes caen cerca de la misma nota esperada, ambos reciben su propio color y desvío. (RF-29.1, RF-29.6)
- **FR-029**: El sistema MUST colorear cada golpe de verde (desvío de hasta 100 ms y pieza esperada), amarillo (más de 100 y hasta 250 ms, pieza esperada) o rojo (más de 250 ms, sin nota esperada a menos de 250 ms o pieza distinta). (RF-29.2, RF-29.3, RF-29.4)
- **FR-030**: El sistema MUST marcar como omitida, con un indicador distinto de los tres colores, la nota esperada sin ningún golpe a 250 ms o menos, una vez pasados 250 ms desde su instante. (RF-29.5)
- **FR-031**: El sistema MUST mostrar en vivo la pieza esperada, el color y el desvío en milisegundos con signo de cada golpe. (RF-30)
- **FR-031a**: Al terminar el Modo Práctica, el sistema MUST dejar visibles en la partitura el color y el desvío de cada golpe y las notas omitidas, hasta que se inicie otra práctica, se edite la partitura o el usuario pulse "Limpiar práctica", que MUST borrar ese feedback. Con el bucle activado, al empezar cada vuelta MUST borrar el feedback de la vuelta anterior, de modo que solo se vea el de la vuelta actual y, al terminar, el de la última. (RF-30.5, RF-30.6, RF-30.7)
- **FR-032**: El sistema MUST permitir regular el volumen de la partitura durante la práctica, desde el silencio total hasta su nivel normal, sin detenerla y de forma independiente del volumen del metrónomo. (RF-31)

**Identificación y servidor**
- **FR-033**: El sistema MUST exigir sesión iniciada con una cuenta de Google antes de dar acceso al editor, la biblioteca y el Modo Práctica, registrar al usuario la primera vez (con su nombre y email), reconocerlo en logins posteriores desde cualquier computadora y permitir cerrar sesión. Cada cuenta MUST tener una sola sesión activa: al iniciar sesión en otra computadora se cierra la anterior, que muestra la pantalla de identificación con el aviso "Tu sesión se cerró porque iniciaste en otra computadora". Al cerrar sesión, MUST mostrar solo la pantalla de identificación y descartar de la memoria la partitura abierta, el historial de Deshacer, la biblioteca, el mapeo MIDI y los umbrales del usuario; el kit puede quedar cargado. (RF-32, RF-33, RF-34, RF-36, RF-36.1, RF-36.2)
- **FR-034**: El sistema MUST mantener la sesión entre cierres de la aplicación hasta que el usuario cierre sesión o se revoque en el servidor, y no permitir iniciar sesión si el sistema no ofrece un almacén seguro para protegerla. Cuando la sesión deja de ser válida, MUST mostrar la identificación con "Tu sesión venció, identifícate de nuevo" (vencida o revocada) o "No se pudo verificar tu sesión, identifícate de nuevo" (credencial alterada o ya usada). (RF-35, RF-35.2)
- **FR-035**: El sistema MUST asociar cada partitura guardada a su dueño y mostrar solo las partituras del usuario con sesión iniciada. (RF-37, RF-38)
- **FR-036**: El sistema MUST mostrar un aviso de "Sin conexión" y bloquear el acceso si al abrir no puede comunicarse con el servidor y no hay sesión guardada; con sesión guardada MUST abrir el editor en modo sin conexión (FR-036a). Además MUST mostrar un aviso visible si la conexión se pierde durante el uso: fijo, sin tapar el editor ni el Modo Práctica, mientras no haya conexión, y que desaparece al volver. (RF-39, RF-40)

**Calidad, seguridad y compatibilidad**
- **FR-036a**: En modo sin conexión el sistema MUST abrir el editor con el borrador local anterior (o una partitura vacía de 1 compás), permitir escribir y editar con el mouse, y deshabilitar con el motivo "Sin conexión" el guardado, la apertura, la eliminación y el listado de la biblioteca. Si el kit no está en memoria, MUST mantener deshabilitados Play, la grabación y el Modo Práctica con el motivo "Sin conexión: el kit no está cargado". (RF-39.3, RF-39.4, RF-39.5)
- **FR-036b**: Mientras no hay conexión, ya sea porque faltaba al abrir la aplicación o porque se cortó durante el uso, el sistema MUST guardar cada cambio en un borrador local cifrado como máximo 5 s después del último. Al volver la conexión MUST subirlo como partitura en curso sin preguntar si la del servidor es la misma de la que partió el borrador, y si difiere MUST preguntar al usuario cuál conservar, descartando la otra. El borrador pertenece a su usuario, nunca se muestra a otro y se borra al subirse, al cerrar sesión o cuando el usuario lo descarta; si la sesión no se puede renovar al reconectar, se conserva cifrado hasta que el mismo usuario vuelva a identificarse. (RF-39.6, RF-39.7, RF-39.8)
- **FR-037**: El sistema MUST programar cada nota y cada click del metrónomo con error menor a 1 ms respecto de su instante teórico, sin error acumulado, entre 30 y 260 BPM. (RNF-01)
- **FR-038**: Un golpe MIDI válido MUST verse en la partitura en menos de 250 ms (p95) y, en Modo Práctica, su feedback visual en menos de 100 ms (p95) con desvío fiel dentro de ±20 ms. (RNF-02, RNF-03)
- **FR-039**: La grilla rítmica MUST representar la negra con 48 unidades enteras, y en cualquier compás y voz las figuras y los silencios MUST completar el compás exactamente. (RNF-04, RNF-05)
- **FR-040**: El editor, la reproducción y el guardado MUST funcionar igual con y sin MIDI. (RNF-07)
- **FR-041**: Cada partitura guardada MUST llevar el número de versión de su formato (hoy, 1) y toda versión nueva MUST abrir sin intervención las partituras de versiones anteriores con el 100 % de sus notas en la misma posición. Todo cambio de formato MUST incluir, antes de publicarse, una partitura de prueba de cada versión anterior publicada y su escenario de migración. (RNF-09, RNF-09.1)
- **FR-042**: La identificación MUST abrirse en una ventana externa a la aplicación y volver a ella al terminar. Cada pedido al servidor MUST ser autenticado, y el servidor MUST rechazar el acceso ante cualquier credencial alterada, vencida o ya usada. (RNF-10, RNF-11, RNF-12)
- **FR-043**: La sesión y el borrador sin conexión de la partitura en curso MUST ser los únicos datos del usuario guardados en la computadora y MUST estar cifrados, nunca en texto plano. (RNF-13)
- **FR-044**: El servidor MUST aplicar el control de acceso: ningún usuario puede leer, modificar, borrar ni crear datos a nombre de otro (partituras, partitura en curso, mapeo MIDI y perfil), con o sin credenciales válidas. (RNF-14)
- **FR-045**: El kit predefinido MUST pesar 3 MB o menos, con samples de licencia redistribuible registrada, estar listo en 5 s o menos (p95) tras el login con 10 Mbps, y la reproducción MUST NOT pedir sonidos a la red. (RNF-15)
- **FR-046**: El autoguardado MUST subir la partitura como máximo 5 s después del último cambio, y la partitura en curso MUST recuperarse en menos de 3 s (p95) tras el login. (RNF-16, RNF-08)
- **FR-047**: La aplicación MUST ejecutarse en Windows, macOS y Linux. (RNF-17)
- **FR-048**: En el entorno de pruebas, la aplicación MUST permitir inyectar golpes MIDI simulados con marca de tiempo conocida y controlar el reloj de audio, para probar sin batería ni audio real los escenarios de grabación, Modo Práctica y tiempos. Esa capacidad MUST NOT estar disponible para el usuario en producción. (RNF-18)

### Key Entities

- **Usuario / Perfil**: Baterista identificado con su cuenta de Google; tiene nombre y email. Es dueño de todos sus datos.
- **Partitura**: Compases, tempo, compás (numerador/denominador) y notas, con número de versión de formato.
- **Compás**: Unidad de la partitura; contiene notas y silencios derivados en dos voces (manos y pies).
- **Nota**: Pieza del kit, posición rítmica en la grilla, figura (con o sin tresillo), articulación.
- **Biblioteca**: Conjunto de partituras guardadas con nombre de un usuario, con tempo, cantidad de compases y fecha de modificación.
- **Partitura en curso**: Única partitura autoguardada de cada usuario, recuperada al reabrir.
- **Mapeo MIDI**: Asignación de notas MIDI (0–127) a piezas del kit; uno por cuenta.
- **Umbrales de articulación**: Par de intensidades (fantasma y acento) que decide la articulación de las notas grabadas; uno por cuenta, con valores por defecto 30 y 100.
- **Kit predefinido**: Conjunto de samples, uno por pieza, igual para todos, con licencia registrada.
- **Toma**: Grabación de golpes sobre N compases con grilla de cuantización y modo de arranque.
- **Golpe**: Evento MIDI con nota, canal, intensidad y marca de tiempo; en práctica recibe un color (verde, amarillo, rojo) y un desvío con signo.
- **Sesión**: Credenciales de acceso del usuario, guardadas cifradas.
- **Borrador sin conexión**: Copia local cifrada de la partitura en curso, escrita mientras no hay conexión; pertenece a un usuario y se sube o se descarta al reconectar.

## Pantallas y Estados

Lista de las pantallas, sus controles y sus estados, derivada de los requisitos anteriores. Sirve de base para el diseño visual que exige la constitución (principio IV); no define colores, tipografías ni disposición.

### 1. Identificación

- **Controles**: botón "Identificarse con Google".
- **Estados**: sin sesión; identificándose (en una ventana externa a la aplicación); "Almacén de claves no disponible" (FR-034); "Sin conexión" sin sesión guardada (FR-036), sin acceso al resto de la aplicación.

### 2. Editor (pantalla principal)

- **Controles de escritura**: pentagrama de percusión; selector de figura (redonda, blanca, negra, corchea, semicorchea) con tresillo activable; selector de articulación (normal, acento, fantasma, flam); selección de nota con clic y Borrar (o tecla Supr); Añadir compás; Quitar el último; Deshacer; Nueva partitura.
- **Controles de transporte**: Play/Detener; bucle; tempo (número entre 30 y 260); compás (numerador 1–16, denominador 2/4/8/16); metrónomo (activar y volumen propio); indicador de posición sobre la partitura.
- **Controles de cuenta y guardado**: Guardar en la biblioteca (con nombre); acceso a la Biblioteca; Cerrar sesión.
- **Estados**: cargando kit ("Cargando kit"); error de kit con "Reintentar"; sin conexión (aviso fijo, modo borrador, biblioteca deshabilitada con "Sin conexión" y kit no cargado); "No se están guardando los cambios"; aviso de notas descartadas por un cambio de tipo de compás; partitura recuperada al abrir.

### 3. Conexión de la batería MIDI

- **Controles**: Conectar batería; listado de entradas; último golpe recibido (nota, canal, intensidad); mapeo MIDI (reasignar cualquier nota 0–127 a una pieza del kit).
- **Estados**: sin permiso o función no disponible ("Web MIDI no está disponible", "Permiso MIDI denegado"); entrada ocupada ("No se pudo abrir la entrada MIDI"); conectada; "Batería desconectada".

### 4. Grabación de toma

- **Controles**: cantidad de compases (1 a 128); grilla de cuantización (semicorchea, corchea de tresillo, corchea, negra); arranque (al primer golpe o con cuenta previa); umbrales de articulación (fantasma y acento); Grabar; Detener (y Esc).
- **Estados**: esperando el primer golpe; en cuenta previa; grabando; terminada (golpes conservados). Grabar y Modo Práctica deshabilitados, con su motivo, si no hay MIDI o el kit no está cargado.

### 5. Modo Práctica

- **Controles**: activar Modo Práctica; cuenta previa opcional; bucle; metrónomo; volumen de la partitura (de silencio total a su nivel normal); Limpiar práctica.
- **Visualización en vivo**: nota esperada destacada; pieza esperada; color de cada golpe (verde, amarillo, rojo); desvío en milisegundos con signo; nota omitida con indicador propio.
- **Estados**: en cuenta previa; practicando; terminada con feedback visible; "Batería desconectada" (práctica detenida).

### 6. Biblioteca

- **Controles**: listado de partituras (nombre, tempo, compases, fecha de modificación); abrir; eliminar.
- **Estados**: biblioteca vacía; error al guardar por nombre vacío, repetido o de más de 300 caracteres.

### 7. Diálogos de confirmación

- Eliminar partitura (RF-26.1).
- Abrir una partitura de la biblioteca con cambios sin guardar en la partitura en curso (FR-026).
- Al reconectar con un borrador distinto de la partitura en curso del servidor: elegir entre el borrador local y la versión del servidor (FR-036b).
- Nueva partitura con cambios sin guardar en la biblioteca (FR-024a).

## Glosario

Cada término tiene un único significado en todo el spec.

- **Compás**: cada una de las unidades en que se divide la partitura (de 1 a 128).
- **Tipo de compás**: la indicación numérica de la partitura, como 4/4 o 3/4 (numerador y denominador).
- **Tiempo**: cada pulso de un compás (tiempo 1, tiempo 2, ...).
- **Paso**: cada posición de la grilla de cuantización (semicorchea, corchea de tresillo, corchea o negra) en la que se ubica un golpe al grabar.
- **Marca de tiempo**: el instante exacto en que llega un golpe MIDI. No es lo mismo que un tiempo del compás.
- **Golpe**: evento MIDI que llega de la batería (nota, canal, intensidad y marca de tiempo).
- **Nota**: figura escrita en la partitura; un golpe grabado pasa a ser una nota.
- **Toma**: grabación de golpes sobre una cantidad de compases elegida.
- **Feedback**: conjunto de lo que el Modo Práctica muestra de cada golpe y de cada nota esperada: color, desvío y nota omitida.
- **Color**: verde, amarillo o rojo asignado a un golpe en el Modo Práctica.
- **Desvío**: diferencia en milisegundos, con signo, entre el golpe y la nota esperada.
- **Partitura en curso**: la única partitura de trabajo de cada usuario, que se autoguarda y se recupera al reabrir la aplicación.
- **Borrador sin conexión**: copia local cifrada de la partitura en curso, escrita mientras no hay conexión.
- **Biblioteca**: conjunto de partituras guardadas con nombre por un usuario.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Un baterista escribe o toca un ejercicio, lo guarda, lo escucha, lo practica con su batería MIDI y, al terminar, sabe en qué golpes se adelantó o se atrasó (métrica de éxito del PRD).
- **SC-002**: Una persona que abre la aplicación por primera vez, sin leer documentación, captura un groove de 2 compases (con el mouse o tocándolo) en menos de 2 minutos. *Validación manual con usuarios, sin prueba automática (objetivo O-4 del PRD).*
- **SC-003**: Un baterista que lee partitura entiende un compás escrito en Drum Sync sin explicación previa. *Validación manual con usuarios, sin prueba automática (objetivo O-1 del PRD).*
- **SC-004**: El 100 % de las partituras guardadas con versiones anteriores del formato se abre sin intervención y con todas sus notas en la misma posición.
- **SC-005**: El desvío de timing mostrado en Modo Práctica difiere del real en 20 ms o menos en el 100 % de los golpes de prueba, y el feedback aparece en menos de 100 ms en el 95 % de los golpes.
- **SC-006**: Ninguna nota ni click de una reproducción de 10 minutos, entre 30 y 260 BPM, se aparta 1 ms o más de su instante teórico.
- **SC-007**: Ningún usuario puede ver, modificar ni borrar datos de otro usuario, ni desde la aplicación ni consultando el servidor directamente.
- **SC-008**: Tras el login, el 95 % de las veces el kit está listo en 5 s o menos y la partitura en curso (hasta 128 compases) es visible en menos de 3 s.
- **SC-009**: Un usuario pierde como máximo 5 s de trabajo si la aplicación se cierra de golpe con conexión.
- **SC-010**: La aplicación llega a la pantalla de identificación tras instalarse en Windows, macOS y Linux.

## Assumptions

- La interfaz está solo en español.
- Los usuarios tienen una batería electrónica con salida USB MIDI para grabar tomas y practicar; sin ella usan el editor y la reproducción.
- La aplicación necesita conexión para identificarse, descargar el kit y usar la biblioteca; no hay copia local de la biblioteca ni del kit. Con sesión guardada, el único uso sin conexión es escribir en un borrador local cifrado de la partitura en curso.
- El tempo lo fija siempre el usuario a mano y el compás es único por partitura.
- El kit predefinido es igual para todos y no se puede modificar ni reemplazar.
- El formato actual de partitura es la versión 1; hoy no hay versiones anteriores publicadas. Cuando exista una versión posterior, cada cambio de formato debe incluir una partitura de prueba de cada versión anterior y su escenario de migración (FR-041).
- Hay un único mapeo MIDI por cuenta, no uno por batería.
- Fuera de alcance, según el PRD: transcripción de audio, IA, importar o exportar archivos, funciones de profesor/alumno, edición simultánea, otros proveedores de identificación, versión web para usuarios finales, notación avanzada, accesibilidad, otros idiomas, mezclador por pieza, rehacer, y hardware que no sea una batería USB MIDI.
- La accesibilidad (lector de pantalla, contraste, navegación por teclado) y los idiomas distintos del español quedan fuera de esta versión por decisión de alcance, no por omisión; se reevalúan en una versión posterior.
- En las pruebas automáticas, el rechazo de credenciales de Google se comprueba con credenciales falsas (firma inválida, destinatario ajeno o vencida); una credencial firmada por Google con destinatario ajeno no se puede reproducir y no se prueba.
- El registro y la auditoría de eventos de seguridad (logins fallidos, credenciales rechazadas, accesos denegados) quedan fuera de la aplicación: se usan los registros del proveedor de identidad y del servidor. Es una decisión explícita, no una omisión.
- Los objetivos O-1 y O-4 del PRD son de producto y no tienen criterio de aceptación automatizado.
