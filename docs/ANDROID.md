# Android y APK

Esta guia resume como esta preparado el proyecto para Android y como generar o reutilizar el APK.

## Estado actual

El proyecto ya incluye:

- configuracion `Capacitor`
- proyecto nativo en `android/`
- soporte PWA
- `APK debug` compilado

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

En tablet Android el dictado tiene mucho mas sentido que en escritorio, especialmente si el ordenador no tiene microfono.

Al abrir la app instalada:

1. concede permiso al microfono
2. prueba el boton `Hablar`

## Notas importantes

- El APK actual es `debug`.
- Sirve para pruebas internas.
- No es todavia una build `release` firmada.
- Para Google Play haria falta generar una build firmada o un `AAB`.
