# Hardening

Medidas activas para reducir fallos silenciosos en el MVP.

## Parser versionado

- Cada `planData` parseado desde PDF guarda `source.parserVersion`.
- La version actual del parser vive en `src/lib/pdf-plan-parser.ts`.
- Si un plan fue parseado con una version anterior y era un PDF real del paciente, se marca como `requiresReupload`.

## Bloqueo seguro

- Cuando `requiresReupload` es `true`, el generador no devuelve una comida.
- En su lugar, muestra un mensaje claro para volver a subir el PDF.
- Esto evita seguir generando con gramos potencialmente incorrectos.

## Admin

- El panel del nutricionista muestra mejor el origen del plan.
- Tambien deja visible si el plan necesita recarga.
- La idea es detectar el problema antes de que llegue al paciente.

## Siguiente blindaje recomendado

- Añadir indicador por familia principal/no principal de `parseado` frente a `fallback`.
- Añadir tests E2E reales con subida de PDF y validacion en interfaz.
- Añadir invalidacion o migracion mas fina por bloque, no solo por version global del parser.
