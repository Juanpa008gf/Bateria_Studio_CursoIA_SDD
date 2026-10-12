# Drum Sync Constitution

## Core Principles

### I. Escenarios Gherkin Automatizados (NON-NEGOTIABLE)
Todo comportamiento del producto MUST describirse como escenarios Gherkin (Given/When/Then)
y MUST estar automatizado con Playwright. Cada spec MUST traer sus escenarios, y esos
escenarios MUST existir como pruebas ejecutables antes de dar la implementación por terminada.
Rationale: los escenarios son el contrato verificable entre la spec y el código.

### II. Commit Solo con Escenarios en Verde
Antes de cada commit al repo se MUST ejecutar la suite completa de escenarios Playwright, no
solo los de la feature actual. Si todos pasan, se puede commitear. Si alguno falla, se MUST
consultar a un humano si el fallo puede dejarse para revisar después; sin su aprobación
explícita NO se commitea. La aprobación se aplica solo a ese commit y el escenario fallido
MUST registrarse como pendiente (por ejemplo en el mensaje del commit).
Rationale: evita regresiones silenciosas y deja la decisión de excepciones en una persona.

### III. Análisis OWASP Top 10 Antes de Commitear
Antes de cada commit se MUST hacer un análisis de seguridad contra el OWASP Top 10 sobre los
cambios. Las vulnerabilidades de severidad crítica o alta MUST corregirse antes del commit.
Las de severidad media o baja SHOULD registrarse como pendientes. La severidad se clasifica con
la metodología OWASP Risk Rating: se estima la probabilidad y el impacto de cada hallazgo
(bajo, medio o alto) y la severidad resulta de combinarlos:
impacto alto con probabilidad media es alta y con probabilidad alta es crítica; impacto medio con
probabilidad alta es alta. Complementa, sin
reemplazar, las reglas de seguridad de AGENTS.md (Row Level Security, `service_role`, `safeStorage`).
Rationale: la app maneja sesiones y datos de usuario en Supabase; la seguridad se verifica de
forma continua y no solo al final.

### IV. Diseño Visual Aprobado Antes de Implementar
Antes de implementar una spec que cambie la interfaz, se MUST mostrar a un humano un diseño
en imagen del look & feel de los cambios, y esperar su aprobación. No se escribe código de
interfaz hasta contar con esa aprobación. Si el diseño cambia durante la implementación,
MUST volver a mostrarse.
Rationale: detectar desacuerdos visuales antes de construir evita retrabajo.

## Restricciones Técnicas

El stack, los comandos y las prohibiciones del proyecto (compatibilidad de partituras
guardadas, independencia de Web MIDI, red fuera del camino del timing, Row Level Security,
sesión cifrada con `safeStorage`) están definidos en `AGENTS.md` y MUST cumplirse. Ante un
conflicto entre este documento y `AGENTS.md`, prevalece la regla más estricta.

## Flujo de Desarrollo y Puertas de Calidad

Orden obligatorio por cada spec:
1. Spec con escenarios Gherkin.
2. Diseño en imagen aprobado por un humano (Principio IV), si hay cambios de interfaz.
3. Automatización de los escenarios en Playwright y su implementación.
4. Antes de cada commit: suite completa en verde (Principio II) y análisis OWASP Top 10
   con críticas y altas corregidas (Principio III).
5. Commit con mensaje en Conventional Commits.

## Governance

Esta constitución prevalece sobre otras prácticas del proyecto. Toda enmienda MUST
documentarse en este archivo, con aprobación del responsable del proyecto y, si afecta a
trabajo existente, un plan de migración. El versionado sigue semver: MAJOR para eliminar o
redefinir principios de forma incompatible, MINOR para agregar un principio o sección o
ampliar guía de forma material, PATCH para aclaraciones y correcciones de redacción. Cada
spec, plan y revisión MUST verificar el cumplimiento de estos principios; cualquier
excepción MUST justificarse por escrito. La guía de desarrollo en ejecución está en `AGENTS.md`.

**Version**: 1.0.1 | **Ratified**: 2026-10-12 | **Last Amended**: 2026-10-12
