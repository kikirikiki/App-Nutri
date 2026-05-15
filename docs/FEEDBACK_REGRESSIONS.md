# Feedback y regresiones protegidas

Este documento registra incidencias reales reportadas durante el uso de Exact Meals y como quedan protegidas.

Objetivo:

- Que cada fallo corregido tenga una referencia estable.
- Que una correccion nueva no destruya una anterior.
- Que el comportamiento esperado sea comprobable por tests, auditoria o checklist manual.
- Que quede claro si el caso esta automatizado o solo documentado.

## Estado general

- Ultima actualizacion: 2026-05-15.
- Tests unitarios actuales: `104`.
- Suite E2E actual: `4`.
- Build web: correcto en la ultima validacion.
- APK debug: reconstruida tras la ultima tanda de correcciones.

## Matriz de feedback

| ID | Fecha | Area | Entrada / situacion reportada | Fallo observado | Comportamiento esperado | Proteccion | Estado |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FB-001 | 2026-04 | Modo pareja | `25g de pasta, pan y pollo` en modo pareja | La app daba 25g a Maria y 50g a Juan, usando 75g totales; tambien podia degradar aceite | La cantidad explicita se trata como stock compartido y se reparte proporcionalmente; la grasa no limitada sigue normal | `src/lib/couple-mode.test.ts` | Corregido |
| FB-002 | 2026-04 | Parser / planes viejos | Planes parseados antes de corregir el parser | Un plan viejo seguia en `localStorage` aunque el parser ya estuviera arreglado | Plan PDF antiguo queda marcado `requiresReupload` y el motor bloquea | `src/lib/plan-storage.test.ts`, `src/lib/scope-lock.test.ts` | Corregido |
| FB-003 | 2026-04 | Alias culinarios | `macarrones` no aparecia en PDF, pero el usuario espera `Pasta` | El motor podia no asociarlo o depender demasiado de Gemini | Alias seguro `macarrones -> Pasta`, solo si la equivalencia base esta confirmada | `keeps culinary aliases but blocks them when the base PDF equivalent only came from fallback` | Corregido |
| FB-004 | 2026-04 | Humus / hummus | `Dime la cantidad de humus que puedo comer` | No daba cantidad clara o intentaba resolver como comida no principal | Si `hummus` aparece literal en el plan, devuelve cantidad; si no aparece, bloquea y pide confirmacion | `answers a direct quantity lookup...`, `does not ask for a non-main family...` | Corregido |
| FB-005 | 2026-04 | Humus / hummus | `humus` | Se llego a generar una comida imposible con legumbres, huevo y aceite | Hummus no se convierte automaticamente en garbanzos/legumbres; se bloquea si no hay equivalencia literal | `does not turn an unknown food like humus into eggs or derived legumes...` | Corregido |
| FB-006 | 2026-04 | No principales | `pollo con arroz` funcionaba en web, pero en app/tablet pedia concretar `Carne blanca` | Web y app tenian datos locales distintos o plan fallback viejo | Si una equivalencia principal es fallback, se bloquea con mensaje de plan local viejo/distinto | `keeps culinary aliases but blocks them...`, docs `HARDENING.md` | Corregido |
| FB-007 | 2026-04 | Android / Gemini | Gemini inactivo o con error en tablet | La app instalada no encontraba backend usando `localhost` o backend apagado | Backend configurable desde `Nutricionista > Ajustes`; usar IP del ordenador o URL HTTPS | Checklist manual + docs `ANDROID_INSTALLATION.md` | Documentado |
| FB-008 | 2026-04 | Android 7.1.7 | Instalacion falla en tablet antigua | Play Protect/instalador/WebView/firma podia bloquear APK | Documentar minSdk, ADB, prueba interna Google Play y conflicto de firma | docs `ANDROID_INSTALLATION.md` | Documentado |
| FB-009 | 2026-04 | UI movil | En movil/tablet no se veia el boton salir en nutricionista | Header ocultaba salida en anchos pequenos | Boton compacto `Salir` visible en movil | Build + revision UI | Corregido |
| FB-010 | 2026-05 | Backend local | `EADDRINUSE 0.0.0.0:5173` al arrancar | Se interpretaba como fallo de app | Significa que el backend ya esta corriendo en ese puerto | Checklist operativo | Documentado |
| FB-011 | 2026-05 | Feedback cliente | No habia forma de reportar fallos con contexto | Los fallos se perdian o llegaban sin datos suficientes | Boton `Reportar fallo`, log local, copiar/exportar JSON, panel `Incidencias` | `src/lib/feedback-log.test.ts` | Corregido |
| FB-012 | 2026-05 | Feedback cliente | Descargar incidencias desde dispositivo | El log quedaba solo dentro de `localStorage` | Boton `Descargar JSON`; archivo `exact-meals-*.json` por flujo normal de descargas | Build + test de serializacion | Corregido |
| FB-013 | 2026-05 | Persistencia | Web y app muestran pacientes/parejas/incidencias distintas | Cada instalacion usa su propio `localStorage` | Documentar que no hay base de datos compartida; siguiente salto recomendado: backend/Supabase | docs + respuesta de arquitectura | Documentado |
| FB-014 | 2026-05 | Comida principal | `Quiero un risotto de gambas y queso` | Reconocia arroz y queso, pero no gambas como proteina | `risotto -> Arroz y derivados`; `gambas/langostinos/camarones -> Pescado blanco` si existe en plan | `maps risotto to arroz and gambas to pescado blanco...` | Corregido |
| FB-015 | 2026-05 | No principal / atun | `bocadillo de atunal natural... aguacate` | Escogia `Atun en aceite de oliva` como si fuera atun en aceite | Si el usuario pide natural, prioriza `Atun al natural` aunque haya typo | `prefers atun al natural over atun en aceite...` | Corregido |
| FB-016 | 2026-05 | No principal / cottage | `tostada de huevos, cottage y jamon de pavo a partes iguales` | Sustituia cottage por `Claras de huevo` | Si cottage no esta en esa subfamilia, bloquea y pide subfamilia compatible | `does not replace requested cottage with claras...` | Corregido |
| FB-017 | 2026-05 | Fuera de plan | `pizza de atun y bacon al cincuenta por ciento` | Podia ignorar pizza/bacon y pedir solo carbohidrato/proteina | Reconoce reparto, pero bloquea pizza/bacon si no son equivalencias operativas del PDF | `blocks unsupported pizza or bacon requests...` | Corregido |

## Reglas de no regresion

Antes de aceptar una nueva correccion del motor:

1. Ejecutar `npm.cmd run test`.
2. Ejecutar `npm.cmd run build`.
3. Si afecta UI, ejecutar o ampliar E2E.
4. Si corrige un feedback nuevo, anadir fila a esta matriz.
5. Si modifica alias, comprobar que:
   - el alias solo apunta a una equivalencia confirmada.
   - no se usa si la equivalencia viene de fallback.
   - no sustituye un alimento pedido por otro no pedido.
6. Si modifica Gemini, comprobar que:
   - Gemini ayuda a interpretar.
   - el motor determinista sigue validando.
   - backend caido no rompe generacion local.

## Tests clave actuales

Archivo principal:

```text
src/lib/scope-lock.test.ts
```

Casos especialmente sensibles:

- `maps common culinary names like raviolis to the pasta equivalent in the plan`
- `maps burrito-style wording to wraps instead of falling back to legumes`
- `accepts close spellings like cuscus and maps them to the couscous equivalent`
- `does not turn an unknown food like humus into eggs or derived legumes in non-main meals`
- `keeps culinary aliases but blocks them when the base PDF equivalent only came from fallback`
- `maps risotto to arroz and gambas to pescado blanco when those base equivalents exist`
- `prefers atun al natural over atun en aceite when the user asks for natural, even with a typo`
- `does not replace requested cottage with claras when cottage belongs to another subfamily`
- `blocks unsupported pizza or bacon requests instead of silently ignoring them`

Modo pareja:

```text
src/lib/couple-mode.test.ts
```

Feedback:

```text
src/lib/feedback-log.test.ts
```

## Casos documentados pero no totalmente automatizados

Estos casos dependen de entorno/dispositivo y deben revisarse manualmente:

- Android 7.1.7 y bloqueo de instalacion.
- Google Play Protect / sideload.
- Diferencia de datos entre web y app por `localStorage` separado.
- Backend Gemini accesible desde tablet usando IP local.
- Boton de descarga JSON en WebView Android antiguo.

## Pendientes recomendados

- Convertir los casos manuales mas importantes a E2E o pruebas de integracion cuando haya backend/base de datos real.
- Anadir sincronizacion compartida para pacientes, parejas, planes e incidencias.
- Mantener esta matriz como fuente de verdad antes de tocar alias, parser o modo pareja.
