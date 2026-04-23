# Changelog

Registro de cambios funcionales relevantes del MVP.

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
