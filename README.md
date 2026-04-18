# Exact Meals

Aplicacion web para traducir planes nutricionales a comidas concretas y exactas a partir de lo que el paciente tiene disponible o de lo que le apetece comer.

El proyecto esta orientado a una nutricionista y sus pacientes. El objetivo del MVP es convertir una pauta nutricional real en una experiencia simple para el paciente: escribir o dictar lo que tiene, y recibir solo los ingredientes y cantidades que debe usar segun su plan.

## Que hace ahora mismo

- Separa interfaz de paciente y de nutricionista.
- Permite cargar planes desde `PDF` o `JSON` en `/admin`.
- Convierte el PDF a un modelo estructurado interno.
- Guarda planes en `localStorage` para trabajar en local sin backend.
- Genera comidas exactas desde el plan real del paciente.
- Incluye un modo `No pensar` derivado del plan real del paciente.
- Incluye una propuesta `Semanal` derivada del plan real del paciente.
- Permite guardar gustos del paciente para personalizar esas propuestas.
- Soporta reglas distintas para:
  - comidas principales
  - comidas no principales
- Usa equivalencias derivadas entre contextos cuando un alimento existe en comida o desayuno y falta en el otro.
- Sugiere grasas compatibles cuando faltan para cerrar una comida.
- Incluye dictado por voz en navegador y soporte nativo dentro del APK Android.
- Se puede instalar como `PWA`.
- Tiene proyecto Android con `Capacitor` y un `APK debug` generado.

## Enfoque funcional

La app ya no trata los planes como una lista plana de recetas. Internamente separa:

- `reglas globales del sistema`
- `resumen diario del paciente`
- `equivalencias de comidas principales`
- `familias de comidas no principales`
- `equivalencias extra`

Esto viene de dos tipos de documentos:

- el PDF de instrucciones generales
- el PDF especifico de cada paciente

## Stack

- React 18
- Vite 5
- TypeScript 5
- Tailwind CSS v3
- Framer Motion
- React Router DOM
- pdf.js
- Vitest
- Playwright
- PWA con `vite-plugin-pwa`
- Android con `Capacitor`

## Estructura principal

```text
src/
  components/   UI principal, generador y resultado
  contexts/     auth y roles
  lib/          parser PDF, almacenamiento local y motor Scope Lock
  pages/        admin, auth, generar, rapido, semana
  test/         setup de tests
  types/        tipos de dominio
android/        proyecto nativo Android con Capacitor
public/         iconos, assets y ejemplo de plan
.cert/          certificado local para HTTPS en localhost
```

## Rutas de la app

- `/` inicio
- `/auth` acceso demo por rol
- `/generar` generador del paciente
- `/rapido` acceso rapido
- `/semana` vista semanal
- `/admin` gestion de pacientes y carga de planes

## Roles

La app separa claramente ambos perfiles:

- `patient`
  - puede generar comidas
  - puede consultar rapido y semana
- `nutritionist`
  - puede gestionar pacientes
  - puede subir PDF o JSON
  - puede actualizar y eliminar pacientes

## Flujo actual

### Nutricionista

1. Entra en `/auth` como nutricionista.
2. Va a `/admin`.
3. Sube un `PDF` o `JSON`.
4. Revisa la vista previa estructurada del plan.
5. Guarda o actualiza el paciente.

### Paciente

1. Entra en `/auth` con el email asociado.
2. Va a `/generar`.
3. Escribe o dicta lo que tiene o lo que le apetece.
4. La app devuelve ingredientes y cantidades del plan.
5. Puede entrar en `/rapido` para elegir opciones cerradas sin pensar.
6. Puede entrar en `/semana` para ver una propuesta semanal basada en su plan y en sus gustos.

## Parser del PDF

El parser intenta representar fielmente el sistema nutricional del documento:

- `comidas principales`
  - metodo del plato
  - equivalencias de carbohidratos, proteinas y grasas
  - verduras como componente obligatorio pero flexible
- `comidas no principales`
  - familias reales del PDF
  - familias por bloques
  - familias de opciones completas

## Scope Lock

El generador usa una logica de validacion para evitar respuestas incoherentes:

- detecta el contexto de la peticion
- intenta mapearla a la familia o equivalencia correcta
- prioriza coincidencias fuertes
- evita confusiones como:
  - `arroz` frente a `tortitas de arroz`
  - `claras de huevo` frente a `huevo entero`
- sugiere grasas compatibles cuando faltan
- puede derivar cantidades entre desayuno y comida cuando el alimento falta en uno de los contextos

## Voz

El campo principal del generador permite dictado por voz.

Notas importantes:

- funciona mejor en Chrome o Edge
- requiere permiso de microfono
- necesita contexto seguro:
  - `https://...`
  - o `https://localhost:5173` en local
- si el dispositivo no tiene microfono, el dictado no podra activarse
- dentro del APK Android usa reconocimiento de voz nativo con Capacitor
- en Android el boton del micro queda activo hasta que el usuario vuelve a pulsarlo
- una segunda pulsacion lo apaga correctamente dentro del APK
- cuando esta escuchando, el boton pasa a estado visual destacado para que sea evidente

## Personalizacion del paciente

Las pantallas `Rapido` y `Semana` se apoyan en una encuesta rapida de gustos del paciente.

Se guardan:

- ingredientes favoritos
- ingredientes que prefiere evitar
- formatos preferidos

Con eso:

- `No pensar` ofrece tarjetas cerradas del plan real, priorizando lo que mas le gusta
- `Semana` genera una propuesta semanal a partir del plan real, pero rotando segun esas preferencias

## Desarrollo local

### Requisitos

- Node.js 18+
- npm

### Instalar dependencias

```powershell
npm.cmd install
```

### Arrancar en local

```powershell
npm.cmd run dev
```

La app se sirve en:

```text
https://localhost:5173
```

Se usa HTTPS local si existe el certificado en `.cert/localhost.pfx`.

### Build

```powershell
npm.cmd run build
```

### Tests

```powershell
npm.cmd run test
```

## PWA

La app esta preparada para instalarse como aplicacion web en tablet o movil.

Incluye:

- `manifest.webmanifest`
- service worker
- iconos Android
- registro automatico del SW

Para instalarla como PWA en Android:

1. publica la app bajo HTTPS
2. abrela en Chrome Android
3. usa `Anadir a pantalla de inicio`

## Android y APK

El proyecto incluye un wrapper nativo Android con Capacitor.

Scripts utiles:

```powershell
npm.cmd run cap:sync
npm.cmd run android:open
npm.cmd run android:copy
npm.cmd run apk:versioned
```

Se ha generado un APK debug en el proyecto local en:

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

Despues de compilarlo, se puede generar una copia versionada con:

```powershell
npm.cmd run apk:versioned
```

La salida versionada queda en:

```text
apk/
```

Y las versiones antiguas se mueven a:

```text
apk/archivo
```

La guia completa de Android esta en `docs/ANDROID.md`.

## Almacenamiento actual

Ahora mismo el MVP funciona en local con `localStorage`.

Esto implica:

- no hay backend real conectado
- no hay persistencia entre dispositivos
- los planes cargados pertenecen al navegador local actual

El proyecto mantiene placeholders para integracion futura con Supabase.

## Estado actual

Lo que esta especialmente bien cubierto en este punto:

- carga de PDF
- vista previa estructurada en admin
- separacion de roles
- generacion basada en plan real
- modo no pensar derivado del plan real
- semana derivada del plan real
- personalizacion por gustos del paciente
- equivalencias derivadas entre contextos
- voz en navegador compatible
- voz nativa dentro del APK Android
- boton de micro persistente y visualmente destacado en tablet
- instalacion PWA
- empaquetado Android con APK debug
- versionado manual del APK con carpeta de archivo

## Proximos pasos recomendados

- conectar almacenamiento real con Supabase
- anadir edge functions para generacion remota
- soportar release APK firmado
- crear build de produccion para Play Store o distribucion privada
- anadir trazabilidad visual minima para cantidades derivadas
- mejorar publicacion web con dominio HTTPS real

## Licencia

Pendiente de definir.
