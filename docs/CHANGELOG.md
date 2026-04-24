# Changelog

Registro de cambios funcionales relevantes del MVP.

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
- Se ha añadido una regresion automatica para asegurar que un plan antiguo no vuelva a colarse como si fuera valido.

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
