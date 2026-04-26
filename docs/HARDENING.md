# Hardening

Medidas activas para reducir fallos silenciosos en el MVP.

## Principio de seguridad funcional

- La app no debe inventar.
- La app no debe convertir una relacion semantica en una cantidad si el PDF no trae esa equivalencia operativa.
- La app debe preferir bloquear y pedir recarga o confirmacion antes que dar una comida aparentemente valida pero falsa.

## Parser versionado

- Cada `planData` parseado desde PDF guarda `source.parserVersion`.
- La version actual del parser vive en `src/lib/pdf-plan-parser.ts`.
- Si un plan fue parseado con una version anterior y era un PDF real del paciente, se marca como `requiresReupload`.

## Bloqueo seguro

- Cuando `requiresReupload` es `true`, el generador no devuelve una comida.
- En su lugar, muestra un mensaje claro para volver a subir el PDF.
- Esto evita seguir generando con gramos potencialmente incorrectos.

## Fallbacks no operativos

- Las equivalencias principales que no se leen con certeza del PDF se marcan como `sourceStatus: "fallback"`.
- El motor no usa esas equivalencias para generar comidas.
- Si el usuario pide un alimento cuya equivalencia esta marcada como fallback, la app bloquea y pide volver a subir el PDF o importar un JSON correcto.
- Este mensaje es especialmente importante en Android, porque web y app no comparten `localStorage` y una tablet puede conservar un plan viejo aunque la web ya este bien.

## Semantica controlada

- Gemini puede ayudar a interpretar intenciones y relaciones de alimentos.
- Gemini no puede introducir alimentos que no esten anclados en el texto original del usuario o en alias seguros.
- Caso protegido:
  - `humus/hummus` puede relacionarse con garbanzos o legumbres.
  - No puede convertirse automaticamente en gramos de `Legumbres cocidas`.
  - Si `hummus` no aparece en el plan con gramos propios, se bloquea.
- Esto evita respuestas falsas como mezclar `Legumbres cocidas`, `Huevo entero` y grasas para una peticion de `humus`.

## Consultas directas de cantidad

- Si el usuario pregunta por cantidad de un alimento concreto, el motor busca ese alimento en:
  - equivalencias principales.
  - familias no principales.
  - extras del plan.
- Si lo encuentra confirmado, devuelve la cantidad exacta.
- Si no lo encuentra, no intenta construir una comida alternativa.
- Puede mostrar alternativas semanticas cercanas, pero sin convertirlas en gramos del alimento pedido.

## Derivaciones limitadas

- Las equivalencias derivadas entre comidas principales y no principales se limitan para evitar cantidades absurdas.
- No se usan derivaciones con ratios incoherentes o mayores que 1 para resolver alimentos no principales.
- Las derivaciones no se usan para puntuar variantes si el alimento no esta reconocido de forma explicita.

## Gemini como apoyo opcional

- El backend seguro expone:
  - `GET /api/gemini/health`
  - `POST /api/gemini/generate`
- La API key no se expone en la APK ni en el frontend.
- La app Android puede configurar la URL del backend desde `Nutricionista > Ajustes`.
- Si el backend esta caido o inaccesible, Gemini puede marcar error, pero el motor local determinista debe seguir respondiendo cuando no necesita Gemini.

## Admin

- El panel del nutricionista muestra mejor el origen del plan.
- Tambien deja visible si el plan necesita recarga.
- La idea es detectar el problema antes de que llegue al paciente.

## Android y planes locales

- Web y Android no comparten el mismo `localStorage`.
- Un plan correcto en la web no garantiza que la tablet tenga el mismo plan cargado.
- Si en web `pollo con arroz` funciona y en tablet pide revisar `Carne blanca`, la causa mas probable es que la tablet conserva un plan local viejo o distinto.
- Accion recomendada:
  - borrar datos de la app o desinstalar.
  - instalar version actual.
  - entrar como nutricionista.
  - volver a subir PDF o importar JSON correcto.

## Instalacion Android

- Android 7.1.7 esta en el limite minimo configurado por la app (`minSdkVersion 24`).
- Aunque sea compatible por SDK, puede fallar la instalacion por:
  - Play Protect o bloqueo de origen desconocido.
  - conflicto de firma con una APK anterior.
  - WebView/Chrome System WebView antiguo.
  - instalador del fabricante.
- Para pruebas tecnicas se recomienda instalar por ADB.
- Para distribucion estable se recomienda Google Play Console con prueba interna y archivo `.aab`.

## Validacion actual

- Tests unitarios: `84`.
- Tests E2E Playwright: `4`.
- Build web: correcto.
- APK release: firmada correctamente con APK Signature Scheme v2.

## Siguiente blindaje recomendado

- Anadir un flujo de exportar/importar plan entre web y Android para evitar diferencias de `localStorage`.
- Anadir E2E especifico de consulta directa de cantidades.
- Anadir test E2E de backend Gemini inaccesible.
- Anadir aviso visible en Admin cuando una tablet tiene un plan con equivalencias fallback.
