# Android installation notes

Notas de instalacion y pruebas para la version Android de Exact Meals.

## Artefactos generados

- APK release para pruebas manuales:

```text
apk\ExactMeals-v0.1.1-release.apk
```

- Android App Bundle para Google Play Console:

```text
apk\ExactMeals-v0.1.1-release.aab
```

- APK debug de Gradle:

```text
android\app\build\outputs\apk\debug\app-debug.apk
```

## Compatibilidad minima

- `minSdkVersion`: 24.
- Android minimo teorico: Android 7.0.
- La tablet reportada usa Android 7.1.7, por lo que entra dentro del minimo teorico.

## Problema observado en Android 7.1.7

En un movil con Android reciente la APK se instala sin problemas.

En una tablet con Android 7.1.7 la instalacion manual puede fallar o comportarse de forma inestable aunque el SDK minimo sea compatible.

Posibles causas:

- Play Protect o bloqueo del instalador por APK externa.
- Instalacion desde una fuente no autorizada.
- Conflicto de firma con una version anterior instalada.
- Datos locales antiguos que sobreviven entre instalaciones.
- WebView/Chrome System WebView antiguo.
- Limitaciones del instalador del fabricante.

## Recomendacion para pruebas

### Opcion 1: ADB

Para pruebas tecnicas, instalar por USB con ADB suele ser mas fiable que enviar la APK a la tablet.

Pasos:

1. Activar `Opciones de desarrollador` en la tablet.
2. Activar `Depuracion USB`.
3. Conectar la tablet por USB.
4. Aceptar la huella RSA.
5. Ejecutar:

```powershell
npm.cmd run android:install:release
```

Ese script:

- busca un dispositivo conectado por ADB.
- desinstala `app.exactmeals.mobile` si existe.
- instala la APK release firmada.

### Opcion 2: Google Play Console

Para distribucion estable, se recomienda subir el `.aab` a una prueba interna de Google Play.

Ventajas:

- Play Protect deja de tratar la app como APK manual desconocida.
- La instalacion se hace desde Play Store.
- Es mas parecido a un flujo real de produccion.
- Reduce problemas con tablets antiguas y permisos de instalador.

## Gemini en Android

Gemini no vive dentro de la APK.

La APK contiene el frontend. La API key vive en el backend Express.

Para que Gemini funcione en tablet:

1. El backend debe estar arrancado en un ordenador o servidor accesible.
2. Tablet y backend deben estar en la misma red o usar una URL publica HTTPS.
3. En la app:

```text
Nutricionista > Ajustes > Backend de Gemini
```

4. Introducir una URL como:

```text
http://192.168.1.253:5173
```

5. Pulsar `Guardar y probar`.

Si el backend esta apagado, Gemini puede aparecer con error. El motor local determinista debe seguir funcionando cuando no necesita Gemini.

## Planes locales en Android

La web y la app Android no comparten `localStorage`.

Esto significa que:

- un plan correcto en la web no garantiza que la tablet tenga el mismo plan.
- la tablet puede conservar un plan viejo.
- una reinstalacion puede no borrar datos si Android conserva almacenamiento.

Si en web `pollo con arroz` funciona y en tablet no:

1. Borrar datos de la app en Android.
2. Desinstalar la app.
3. Instalar la version actual.
4. Entrar como nutricionista.
5. Volver a subir el PDF o importar el JSON correcto.

## Validacion reciente

- Tests unitarios: `84`.
- Tests E2E Playwright: `4`.
- Build web: correcto.
- APK release: firmada correctamente con APK Signature Scheme v2.
