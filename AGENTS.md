# AGENTS.md — Drum Sync

## Propósito
Drum Sync: escribir, escuchar y practicar partituras de batería con una batería MIDI.

## Stack
- React + TypeScript sobre Vite, con VexFlow para la notación.
- Electron como cliente de escritorio; Electron Forge solo para empaquetar.
- La app se distribuye como aplicación de escritorio Electron. Para desarrollo, el cliente debe poder correr también en Chrome o Edge en localhost.
- Supabase como backend (Auth con Google, Postgres con Row Level Security, Storage para los sonidos), vía `supabase-js`. No hay servidor propio!
- Una sola base de datos: la Postgres de Supabase, para usuarios, partituras y configuración. La app no tiene base local.
- Vitest para los tests unitarios y Playwright con `playwright-bdd` para los escenarios Gherkin (`tests/features`); npm como gestor de paquetes.
- Las versiones las fija `package.json`. Node en versión LTS.

## Cómo correr
```bash
npm install
npm run dev        # Vite en http://localhost:5174
npx electron .     # en otra terminal, con Vite ya levantado
npm test           # Vitest (unitarias)
npm run test:e2e   # escenarios Gherkin con Playwright, en navegador y en Electron
npm run verify     # Vitest + todos los escenarios: paso previo a cada commit
```

## Qué NO hacer
- No romper las partituras guardadas por versiones anteriores: cada cambio de formato tiene que migrarlas solo, sin perder notas.
- No hacer que el editor ni la reproducción dependan de Web MIDI: si no está o se deniega el permiso, solo se deshabilitan la grabación y el Modo Práctica (RNF-07).
- No poner la red en el camino del timing: la reproducción, la grabación y el Modo Práctica usan sonidos ya decodificados en memoria, nunca piden nada al servidor (RNF-15).
- No crear una tabla ni un bucket con datos de usuario sin Row Level Security, y no incluir nunca la clave `service_role` en la app (RNF-14).
- No guardar en la computadora más que la sesión y el borrador sin conexión de la partitura en curso (RF-39.6), siempre cifrados con `safeStorage` (RNF-13).
- No commitear sin `npm run verify` en verde y sin el análisis OWASP Top 10 de los cambios; si falla un escenario, consultar a una persona antes de decidir (constitución, principios II y III).
- No implementar interfaz sin el diseño visual aprobado por una persona (constitución, principio IV).
