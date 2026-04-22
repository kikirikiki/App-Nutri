# Android y APK

Esta guia resume como esta preparado el proyecto para Android y como generar o reutilizar el APK.

## Estado actual

El proyecto ya incluye:

- configuracion `Capacitor`
- proyecto nativo en `android/`
- soporte PWA
- `APK debug` compilado
- salida versionada del APK
- reconocimiento de voz nativo en Android

Ruta del APK debug actual:

`app-debug.apk`

## Requisitos

- Node.js
- npm
- Android Studio instalado
- Android SDK
- Java 21 incluido en Android Studio

Variables usadas en esta maquina:

```text
JAVA_HOME=C:\Program Files\Android\Android Studio\jbr
ANDROID_HOME=C:\Users\ingen\AppData\Local\Android\Sdk
ANDROID_SDK_ROOT=C:\Users\ingen\AppData\Local\Android\Sdk
```

El proyecto Android apunta al SDK mediante:

`android/local.properties`

## Flujo de compilacion

Desde la raiz del proyecto:

```powershell
npm.cmd run android:assemble
```

Ese script:

- usa el Java 21 embebido en Android Studio
- hace `build` web
- hace `cap sync android`
- compila el `debug APK`

Si quieres hacerlo a mano, el flujo equivalente es:

```powershell
npm.cmd run build
npx.cmd cap sync android
cd android
.\gradlew.bat assembleDebug
```

## Salida esperada

El APK debug queda en:

```text
android\app\build\outputs\apk\debug\app-debug.apk
```

Despues de generar el debug APK, crea la copia versionada con:

```powershell
npm.cmd run apk:versioned
```

La salida versionada queda en:

```text
apk\ExactMeals-v0.1.0-debug.apk
```

Las versiones anteriores se mueven a:

```text
apk\archivo\
```

## Instalar en una tablet Android

Opciones sencillas:

- pasar el `.apk` por Google Drive
- copiarlo por cable
- enviarlo por almacenamiento compartido

Luego en la tablet:

1. descarga o abre el archivo
2. permite instalar desde origenes desconocidos si Android lo pide
3. instala la app

## Microfono en Android

En tablet Android el dictado tiene mucho mas sentido que en este PC, especialmente si el ordenador no tiene microfono.

Al abrir la app instalada:

1. concede permiso al microfono
2. prueba el boton `Hablar`

Comportamiento actual del boton:

- una pulsacion inicia la escucha
- el boton queda en estado activo hasta que el usuario vuelve a pulsarlo
- una segunda pulsacion detiene la escucha de verdad
- la app evita reactivaciones automaticas no deseadas del motor nativo
- el estilo del boton cambia a un estado visual mucho mas evidente mientras escucha
- el APK usa reconocimiento nativo mediante `@capacitor-community/speech-recognition`, no solo la API web

## Notas importantes

- El APK actual es `debug`.
- Sirve para pruebas internas.
- No es todavia una build `release` firmada.
- Para Google Play haria falta generar una build firmada o un `AAB`.
- La version del archivo se toma de `package.json`.
- La version Android visible tambien se refleja en `android/app/build.gradle`.
- La compilacion Android queda alineada con `Java 21` usando `C:\Program Files\Android\Android Studio\jbr`.

## Archivos implicados

- `capacitor.config.ts`
- `vite.config.ts`
- `src/main.tsx`
- `android`
- `scripts/assemble-android-debug.ps1`
