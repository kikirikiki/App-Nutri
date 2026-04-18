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

## Requisitos

- Node.js
- npm
- Java 17
- Android SDK

Variables usadas en la maquina de desarrollo:

```text
JAVA_HOME=C:\Program Files\Android\Android Studio\jbr
ANDROID_HOME=C:\Users\ingen\AppData\Local\Android\Sdk
ANDROID_SDK_ROOT=C:\Users\ingen\AppData\Local\Android\Sdk
```

## Flujo de compilacion

Desde la raiz del proyecto:

```powershell
npm.cmd run build
npx.cmd cap sync android
```

Luego:

```powershell
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
