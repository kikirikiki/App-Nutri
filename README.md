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
- Compacta la `Semana` para reutilizar mas ingredientes y no disparar la cesta de la compra.
- Genera una `lista de la compra` desde la semana y agrupa ingredientes repetidos.
- Soporta `modo pareja` cuando dos pacientes tienen planes estructuralmente compatibles.
- En modo pareja mantiene la misma comida exacta para ambos y separa cantidades por persona.
- Soporta reglas distintas para:
  - comidas principales
  - comidas no principales
- Usa equivalencias derivadas entre contextos cuando un alimento existe en comida o desayuno y falta en el otro.
- Sugiere grasas compatibles cuando faltan para cerrar una comida.
- Puede usar un fallback opcional con `Gemini` solo para interpretar frases ambiguas o platos compuestos, manteniendo las cantidades en el motor determinista.
- Incluye dictado por voz en navegador y soporte nativo dentro del APK Android.
- Se puede instalar como `PWA`.
- Tiene proyecto Android con `Capacitor` y un `APK debug` generado.

## Documentacion viva

Este repositorio se esta documentando de forma incremental. La idea es que cada cambio funcional relevante quede reflejado en GitHub para que la documentacion no se desincronice de la app.

Cuando se hagan cambios importantes, se actualizaran como minimo:

- este `README.md`
- la guia de Android si afecta al APK o al comportamiento movil
- y las secciones de flujo o estado actual cuando cambie la experiencia real del producto

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
3. Puede usar tres vistas:
   - `Pacientes`
   - `Parejas`
   - `Clientes activos`
4. Sube un `PDF` o `JSON`.
5. Revisa la vista previa estructurada del plan.
6. Guarda o actualiza el paciente.
7. Si dos pacientes tienen planes compatibles, puede crear una pareja para que coman juntos.

### Paciente

1. Entra en `/auth` con el email asociado.
2. Va a `/generar`.
3. Escribe o dicta lo que tiene o lo que le apetece.
4. La app devuelve ingredientes y cantidades del plan.
5. Puede entrar en `/rapido` para elegir opciones cerradas sin pensar.
6. Puede entrar en `/semana` para ver una propuesta semanal basada en su plan y en sus gustos.
7. Si su nutricionista le ha vinculado una pareja compatible, puede cambiar entre modo individual y modo pareja.

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

Archivos clave:

- `src/lib/pdf-plan-parser.ts`
- `src/lib/scope-lock.ts`
- `src/lib/plan-storage.ts`
- `src/types/domain.ts`

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

## Fallback con Gemini

La app ya viene preparada para una capa opcional de interpretacion con `Gemini`.

Importante:

- `Gemini` no calcula gramos
- `Gemini` no elige cantidades
- `Gemini` no puede inventar ingredientes fuera del plan
- solo se usa para traducir lenguaje natural ambiguo a una intencion estructurada

Arquitectura:

1. primero corre el motor local determinista
2. si detecta baja confianza o un caso ambiguo, puede llamar a `Gemini`
3. `Gemini` devuelve una intencion estructurada en JSON
4. la app reconstruye un prompt canonico
5. el motor determinista vuelve a resolver la comida y calcular cantidades

Casos pensados para ese fallback:

- platos compuestos como `tortilla de atun`
- nombres muy cotidianos o poco tecnicos
- frases con formato culinario ambiguo
- casos donde el motor local no cierra bien la intencion

Configuracion local:

1. copia `.env.example` a `.env`
2. rellena la clave:

```text
GEMINI_API_KEY=tu_clave
GEMINI_MODEL=gemini-2.5-flash-lite
```

La app no llama a Gemini directamente desde cliente. El flujo es:

- frontend -> `/api/gemini/health`
- frontend -> `/api/gemini/generate`
- backend local -> Gemini con `GEMINI_API_KEY`

Archivos clave:

- `src/lib/gemini-intent.ts`
- `src/lib/gemini-intent.test.ts`

Nota:

- la clave solo vive en backend
- el badge ya distingue `Gemini inactivo`, `Gemini configurado`, `Gemini operativo` y `Gemini con error`
- la generacion semantica real pasa por backend
- si Gemini falla, la app intenta seguir resolviendo con el motor local cuando puede

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

## Modo pareja

La app ya soporta una primera version operativa del modo pareja.

Condiciones:

- ambos pacientes deben tener planes con la misma estructura
- la pareja se crea desde `Admin`
- el paciente puede alternar entre:
  - `Comer por separado`
  - `Comer con su pareja`

Comportamiento actual:

- `Generar`
  - mantiene la misma comida exacta para ambos
  - muestra cantidades del paciente actual
  - muestra cantidades de la pareja
  - muestra el total para cocinar
- `Rapido`
  - mantiene la misma opcion cerrada para ambos
  - cambia solo las cantidades
- `Semana`
  - proyecta la misma semana sobre el segundo plan
  - consolida la lista de la compra para ambos

Archivos clave:

- `src/lib/couple-mode.ts`
- `src/components/CoupleModePanel.tsx`
- `src/pages/AdminPage.tsx`

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

Archivos clave:

- `vite.config.ts`
- `src/main.tsx`
- `public/icons`

Para instalarla como PWA en Android:

1. publica la app bajo HTTPS
2. abrela en Chrome Android
3. usa `Anadir a pantalla de inicio`

## Android y APK

El proyecto incluye un wrapper nativo Android con Capacitor.

Archivos clave:

- `capacitor.config.ts`
- `android`

Scripts utiles:

```powershell
npm.cmd run cap:sync
npm.cmd run android:open
npm.cmd run android:copy
npm.cmd run android:assemble
npm.cmd run apk:versioned
```

Para compilar Android de forma consistente en esta maquina, el flujo recomendado es:

```powershell
npm.cmd run android:assemble
```

Ese script usa el `Java 21` embebido en Android Studio:

```text
C:\Program Files\Android\Android Studio\jbr
```

Se ha generado un APK debug en:

`app-debug.apk`

Despues de compilarlo, se puede generar una copia versionada con:

```powershell
npm.cmd run apk:versioned
```

La salida versionada queda en:

`apk`

Y las versiones antiguas se guardan en:

`apk/archivo`

La guia completa de Android esta en:

`docs/ANDROID.md`

## Almacenamiento actual

Ahora mismo el MVP funciona en local con `localStorage`.

Esto implica:

- no hay backend real conectado
- no hay persistencia entre dispositivos
- los planes cargados pertenecen al navegador local actual

El proyecto mantiene placeholders para integracion futura con Supabase en:

- `src/integrations/supabase/client.ts`
- `src/integrations/supabase/types.ts`

## Estado actual

Lo que esta especialmente bien cubierto en este punto:

- carga de PDF
- vista previa estructurada en admin
- separacion de roles
- generacion basada en plan real
- modo no pensar derivado del plan real
- semana derivada del plan real
- semana compactada para reducir ingredientes distintos
- lista de la compra derivada de la semana
- personalizacion por gustos del paciente
- modo pareja con la misma comida y cantidades separadas
- admin con vista de pacientes, parejas y clientes activos
- equivalencias derivadas entre contextos
- voz en navegador compatible
- voz nativa dentro del APK Android
- boton de micro persistente y visualmente destacado en tablet
- fallback semantico de Gemini por backend, sin exponer la API key en cliente
- estado visible de Gemini: configurado, operativo o con error
- instalacion PWA
- empaquetado Android con APK debug
- versionado manual del APK con carpeta de archivo
- compilacion Android alineada con Java 21 desde Android Studio

## Cambios recientes

- Se ha reforzado la interpretacion de platos compuestos tipo:
  - `tortilla de atun`
  - `revuelto con pavo`
  - `omelette con salmon`
  - `wrap proteico`
  - `burrito de pollo`
- En esos casos la app ya prioriza una base de huevo o un formato `wrap/fajitas` sin duplicar carbohidratos ni meter opciones genericas raras del mismo bloque.
- Se ha ampliado la suite de regresion con casos reales de `Juan` y `Maria`.
- La bateria actual cubre parser, motor principal, regresion y fallback semantico con un total de `33 tests` verdes.
- Se ha preparado una capa opcional de `Gemini` como fallback semantico para frases ambiguas o platos compuestos:
  - solo interpreta la intencion
  - no calcula gramos
  - no inventa ingredientes
  - siempre devuelve el control al motor determinista
- Se ha movido la integracion de Gemini a backend local para desarrollo:
  - la key vive solo en servidor
  - el frontend consulta `/api/gemini/health`
  - la generacion semantica pasa por `/api/gemini/generate`
- Se ha creado un flujo de compilacion Android estable con `npm.cmd run android:assemble` usando el Java 21 embebido en Android Studio.

## Proximos pasos recomendados

- conectar almacenamiento real con Supabase
- anadir edge functions para generacion remota
- soportar release APK firmado
- crear build de produccion para Play Store o distribucion privada
- anadir trazabilidad visual minima para cantidades derivadas
- mejorar publicacion web con dominio HTTPS real

## Licencia

Pendiente de definir.
