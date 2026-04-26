# Changelog

Registro de cambios funcionales relevantes del MVP.

## 2026-04-26

### Motor de generacion y fidelidad al plan

- Se corrigio un fallo grave por el que `humus` podia acabar convertido en una comida no principal con `Legumbres cocidas`, `Huevo entero` y cantidades imposibles.
- La app ahora distingue entre una relacion semantica y una equivalencia operativa:
  - `humus/hummus` puede relacionarse semanticamente con garbanzos o legumbres.
  - No se convierte automaticamente en gramos de garbanzos o legumbres cocidas.
  - Si `hummus` no aparece literalmente en el plan con gramos propios, el motor bloquea y pide confirmacion del nutricionista.
- Se anadio una ruta de consulta directa de cantidades:
  - si el usuario pregunta "cuanto puedo comer de X" y X esta en el plan, se devuelve la cantidad.
  - si X no esta en el plan, se bloquea sin intentar construir una comida.
- Se reforzo el guardarrail de Gemini:
  - Gemini puede ayudar a interpretar intencion.
  - Gemini no puede introducir alimentos no anclados en el texto original del usuario.
  - conversiones como `humus -> huevos` o `humus -> legumbres con gramos` quedan bloqueadas.
- Se limito el uso de equivalencias derivadas en comidas no principales para evitar cantidades absurdas al cruzar familias o bloques.
- Se cambio el mensaje cuando una equivalencia principal esta marcada como fallback:
  - antes parecia que faltaba concretar un alimento.
  - ahora indica que el plan local necesita volver a cargarse o importar el JSON correcto.

### Modo pareja

- Se corrigio el reparto de cantidades explicitas como stock compartido.
- Ejemplo validado:
  - si hay `25g de pasta` para dos personas, no se asignan 25g a cada una.
  - se reparte proporcionalmente entre los planes de ambos pacientes.
- Se mantiene la grasa normal cuando el usuario no la ha limitado de forma explicita.

### Alias culinarios

- Se reforzaron alias seguros como:
  - `macarrones` -> `Pasta`
  - `cuscus` -> `Couscous`
  - `burrito/wrap` -> `Fajitas / Wraps`
  - `pechuga/pechugas` -> `Carne blanca`
- Los alias solo se usan si la equivalencia base esta confirmada en el plan.
- Si la equivalencia viene de fallback no operativo, el motor bloquea y pide revisar el plan.

### Gemini y backend

- La API key de Gemini sigue fuera del frontend.
- La app Android puede configurar la URL del backend seguro desde `Nutricionista > Ajustes`.
- Se anadio soporte para backend local en red privada, por ejemplo:

```text
http://192.168.1.253:5173
```

- El backend acepta llamadas CORS desde la app instalada.
- Si Gemini no esta disponible porque el backend esta apagado o inaccesible, la generacion local determinista no debe romperse.
- El badge de Gemini puede indicar error, pero el motor local debe seguir funcionando cuando no necesita Gemini.

### Android

- Se genero APK `release` firmada localmente para pruebas.
- Se genero tambien un `.aab` para prueba interna en Google Play Console.
- Se documenta que Android 7.1.7 puede dar problemas instalando APKs modernas por instalador, Play Protect, WebView, firma o sideload.
- Se recomienda:
  - usar ADB para pruebas directas.
  - usar Google Play Console/prueba interna para distribucion estable.
  - borrar datos o desinstalar versiones anteriores cuando haya conflicto de firma o planes locales antiguos.

### UI movil

- En la vista movil de nutricionista se anadio un boton compacto de `Salir` visible en el header.
- Antes el boton solo aparecia en anchos medianos o escritorio, dejando a la nutricionista sin salida clara en movil/tablet estrecha.

### E2E y validacion

- Se anadio suite E2E con Playwright para cubrir flujos reales:
  - generacion con alias confirmado.
  - bloqueo de alias cuando la equivalencia es fallback.
  - modo pareja con stock compartido.
  - smoke completo: nutricionista crea pacientes/pareja y paciente usa generar, rapido y semana.
- Validacion final de esta tanda:
  - `84 tests` unitarios verdes.
  - `4 E2E` verdes.
  - build web correcto.
  - APK release firmada correctamente.

## 2026-04-24

### Hallazgos

- Los planes guardados en local podian seguir usando una lectura antigua del PDF aunque el parser ya se hubiera corregido.
- Eso explicaba parte de las diferencias entre lo que hacia la web/app y lo que ya estaba arreglado en el codigo actual.
- El parser principal sigue teniendo heuristicas y fallbacks: si una lectura vieja se conserva, el motor puede trabajar sobre datos ya desfasados.
- La cobertura de tests era buena a nivel de motor, pero no habia una barrera explicita para bloquear planes parseados con una version antigua.

### Soluciones

- Se ha introducido versionado del parser dentro de `planData.source`.
- Los planes parseados desde PDF con una version antigua del parser quedan marcados como `requiresReupload`.
- El motor de generacion bloquea esos planes y pide volver a subir el PDF, en lugar de seguir generando con datos potencialmente inconsistentes.
- La interfaz de nutricionista muestra ahora mejor el origen del plan, la version del parser y si requiere recarga.
- Se ha anadido una regresion automatica para asegurar que un plan antiguo no vuelva a colarse como si fuera valido.

### Objetivo del blindaje

- Priorizar seguridad funcional sobre falsa sensacion de que el plan esta bien cargado.
- Evitar fallos silenciosos con Maria, Juan o futuros pacientes cuando cambie el parser.
- Forzar reparseo cuando haga falta, en vez de reutilizar lecturas viejas dudosas.

## 2026-04-23

### Motor de comidas principales

- Se ha reforzado la interpretacion de ingredientes escritos con gramos.
- Los gramos indicados por el paciente se usan como pista fuerte, pero el plan nutricional sigue siendo la regla principal.
- Si la cantidad cabe dentro de la equivalencia del plan, se usa tal cual.
- Si la cantidad supera la equivalencia del plan, se capa o se escala para no romper el plan.
- Si un alimento aparece en varios bloques del plan, la app lo puede asignar al bloque donde mas sentido tenga para cerrar la comida.

Caso representativo validado:

```text
90g de arroz
200g de alubias
pollo
```

El plan contempla `Legumbres cocidas` tanto como hidrato como proteina.

Resultado esperado:

- `Arroz y derivados` cubre hidratos.
- `Legumbres cocidas` cubre proteina.
- No se anade `Carne blanca` si la proteina ya queda cubierta por las legumbres.
- Las cantidades nunca superan las equivalencias del plan.

### Android

- Se ha generado una nueva APK debug con estos cambios.
- Ruta local:

```text
apk\ExactMeals-v0.1.1-debug.apk
```

### Validacion

Comandos ejecutados:

```powershell
npm.cmd run test
npm.cmd run build
npx.cmd cap sync android
.\gradlew.bat assembleDebug
npm.cmd run apk:versioned
```

Resultado:

- `36 tests` verdes.
- Build web correcto.
- APK debug generada correctamente.
