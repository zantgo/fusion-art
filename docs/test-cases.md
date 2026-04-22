# Casos de prueba – FusionArt IA (Prototipo)

## 1. Introducción

Este documento define los casos de prueba para validar el correcto funcionamiento del prototipo. Se dividen en tres categorías:

- **Pruebas unitarias**: Validan componentes aislados (utilidades, repositorios, casos de uso).
- **Pruebas de integración**: Validan la interacción entre capas (ViewModel + Repositorio, API real simulada).
- **Pruebas de UI (manuales)**: Validan el flujo completo en un dispositivo o emulador.

El objetivo es garantizar que el prototipo cumple con los requisitos funcionales y no funcionales definidos.

---

## 2. Pruebas unitarias

### 2.1. Utilidad de redimensionamiento (`ImageResizer`)

| ID  | Nombre                         | Descripción                                                                 | Entrada                                                       | Resultado esperado                                                   |
| --- | ------------------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------- |
| UT-01 | Redimensionar imagen más grande | Una imagen de 2048x2048 píxeles.                                            | Bitmap de 2048x2048, config: max 1024x1024                    | Bitmap de 1024x1024 (manteniendo relación de aspecto).               |
| UT-02 | Redimensionar imagen más pequeña | Una imagen de 800x600 píxeles.                                              | Bitmap de 800x600, config: max 1024x1024                      | Bitmap sin cambios (800x600).                                        |
| UT-03 | Compresión con calidad 85%      | Imagen JPEG de 1 MB.                                                        | Bitmap, calidad 85                                            | Archivo de salida menor al original, mayor a 70% de calidad visual. |
| UT-04 | Formato no soportado (opcional) | Pasar un archivo que no es imagen (ej. texto).                              | URI de un archivo .txt                                         | Lanza excepción `IllegalArgumentException`.                          |

### 2.2. Conversión URI a Base64

| ID  | Nombre                         | Descripción                                                                 | Entrada                                                       | Resultado esperado                                                   |
| --- | ------------------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------- |
| UT-05 | Conversión correcta             | URI de una imagen JPEG válida.                                              | Uri con contenido JPEG                                       | String Base64 sin prefijo, no vacío.                                 |
| UT-06 | URI inválida                   | URI que apunta a un recurso inexistente.                                    | Uri ("content://...") no existente                            | Lanza excepción `FileNotFoundException`.                             |

### 2.3. Caso de uso `FusionImageUseCase`

| ID  | Nombre                         | Descripción                                                                 | Entrada (mock)                                              | Resultado esperado                                                   |
| --- | ------------------------------ | --------------------------------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------- |
| UT-07 | Fusión exitosa                 | El repositorio devuelve un Bitmap válido.                                    | URIs de fondo y primer plano                                 | `FusionResult.Success(bitmap)`                                       |
| UT-08 | Error de red                   | El repositorio lanza `IOException`.                                         | URIs válidas                                                | `FusionResult.Error` con mensaje "No hay conexión a internet".       |
| UT-09 | Error de API (401)             | El repositorio lanza excepción con código 401.                              | URIs válidas                                                | `FusionResult.Error` con mensaje "Error de autenticación".           |
| UT-10 | Timeout                        | El repositorio lanza `TimeoutCancellationException`.                        | URIs válidas                                                | `FusionResult.Error` con mensaje "Tiempo de espera agotado".         |

### 2.4. Caso de uso `SaveImageUseCase`

| ID  | Nombre                         | Descripción                                                                 | Entrada (mock)                                              | Resultado esperado                                                   |
| --- | ------------------------------ | --------------------------------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------- |
| UT-11 | Guardado exitoso               | MediaStore permite insertar.                                                | Bitmap válido                                               | `Result.Success(Uri)`                                                |
| UT-12 | Error de escritura             | MediaStore lanza excepción (disco lleno).                                   | Bitmap válido                                               | `Result.Error` con mensaje "No se pudo guardar la imagen".           |

---

## 3. Pruebas de integración

### 3.1. ViewModel + Repositorio (con API real o mock)

| ID  | Nombre                         | Escenario                                                                 | Pasos                                                                                     | Resultado esperado                                                       |
| --- | ------------------------------ | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| IT-01 | Seleccionar imágenes           | Usuario selecciona fondo y primer plano.                                  | 1. Llamar a `selectBackground(uri1)`<br>2. Llamar a `selectForeground(uri2)`               | El estado `FusionState` sigue siendo `Idle`. El botón Fusionar se habilita (observable). |
| IT-02 | Fusión exitosa (mock API)      | Simular respuesta exitosa de la API.                                      | 1. Ambas imágenes seleccionadas.<br>2. Llamar a `fuse()`.                                    | El estado cambia: `Loading` → `Success(bitmap)`.                         |
| IT-03 | Fusión con error de red (mock) | Simular `IOException`.                                                    | Mismo flujo, pero el repositorio lanza excepción.                                         | El estado cambia: `Loading` → `Error("No hay conexión...")`.             |
| IT-04 | Limpiar imágenes               | Después de seleccionar ambas, llamar a `clear()`.                         | 1. Seleccionar fondo y primer plano.<br>2. `clear()`.                                      | El estado sigue `Idle`. El botón Fusionar se deshabilita.                |
| IT-05 | Guardar imagen (con MediaStore real) | Después de fusión exitosa, guardar en galería.                      | 1. Fusión exitosa.<br>2. Llamar a `saveImage()`.                                           | Snackbar de éxito. La imagen aparece en la galería del dispositivo.     |

### 3.2. API real (Gemini) – prueba de integración manual

| ID  | Nombre                         | Escenario                                                                 | Pasos                                                                                     | Resultado esperado                                                       |
| --- | ------------------------------ | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| IT-06 | Llamada real a Gemini          | Usar imágenes de prueba (fondo: paisaje, primer plano: persona).          | 1. Configurar API Key válida.<br>2. Ejecutar `FusionImageUseCase` con URIs reales.         | Recibir `FusionResult.Success` con Bitmap no nulo. La fusión debe ser coherente (iluminación, sombras). |

---

## 4. Pruebas de UI (manuales)

Estas pruebas se ejecutan en un dispositivo físico o emulador con Android 10+.

### 4.1. Flujo principal

| ID  | Escenario                          | Pasos                                                                                     | Resultado esperado                                                                 |
| --- | ---------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| UI-01 | Selección de fondo                 | 1. Tocar "Seleccionar fondo".<br>2. Elegir una imagen de la galería.                      | La miniatura se muestra en la tarjeta de fondo. El botón de ese selector cambia a "Cambiar fondo". |
| UI-02 | Selección de primer plano          | 1. Tocar "Seleccionar primer plano".<br>2. Elegir otra imagen.                            | Miniatura en la tarjeta de primer plano. El botón "Fusionar" se habilita (color primario). |
| UI-03 | Fusión exitosa                     | 1. Tras UI-02, tocar "Fusionar".<br>2. Esperar el indicador de carga.<br>3. Esperar resultado. | Aparece la pantalla de resultado con la imagen fusionada.                        |
| UI-04 | Guardar imagen                     | En pantalla de resultado, tocar "Guardar en galería".                                      | Snackbar "Imagen guardada en galería". La imagen aparece en la app de Galería.     |
| UI-05 | Nueva fusión                       | En pantalla de resultado, tocar "Nueva fusión".                                            | Vuelve a MainScreen con ambas tarjetas vacías. El botón "Fusionar" deshabilitado. |
| UI-06 | Limpiar todo                       | En MainScreen con al menos una imagen, tocar "Limpiar todo".                               | Ambas tarjetas se vacían. Botón "Fusionar" deshabilitado.                         |
| UI-07 | Cambiar una imagen ya seleccionada | 1. Seleccionar fondo.<br>2. Tocar "Cambiar fondo".<br>3. Elegir otra imagen.              | La miniatura se actualiza. No se pierde la otra imagen seleccionada.              |

### 4.2. Manejo de errores (UI)

| ID  | Escenario                          | Pasos                                                                                     | Resultado esperado                                                                 |
| --- | ---------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| UI-08 | Sin conexión a internet            | 1. Desactivar WiFi/datos.<br>2. Seleccionar imágenes.<br>3. Tocar "Fusionar".              | Snackbar: "No hay conexión a internet". No se muestra overlay de carga. El botón se rehabilita. |
| UI-09 | Tiempo de espera (simulado)        | (Modificar código para delay >15s) o usar red muy lenta.                                  | Snackbar: "Tiempo de espera agotado". El botón se rehabilita.                     |
| UI-10 | API Key inválida                   | Configurar una clave inválida en `local.properties`.                                      | Snackbar: "Error de autenticación. La clave de API no es válida".                 |
| UI-11 | Seleccionar archivo no imagen (si el selector lo permite) | Elegir un archivo .txt o .pdf. (Raro, pero puede ocurrir con gestores de archivos). | Snackbar: "No se pudo cargar la imagen. Elige otra". La tarjeta no se actualiza. |
| UI-12 | Rotación de pantalla durante carga | 1. Iniciar fusión.<br>2. Rotar el dispositivo antes de que termine.                       | El overlay de carga se mantiene. Al finalizar, se muestra ResultScreen correctamente. Las URIs no se pierden. |

### 4.3. Pruebas de permisos

| ID  | Escenario                          | Pasos                                                                                     | Resultado esperado                                                                 |
| --- | ---------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| UI-13 | Permiso de lectura denegado (Android 13+) | 1. Instalar app.<br>2. Denegar permiso `READ_MEDIA_IMAGES` cuando se solicite.<br>3. Intentar seleccionar imagen. | La app debe mostrar un diálogo explicativo y volver a solicitar el permiso (o redirigir a ajustes). |

---

## 5. Pruebas de rendimiento (no funcionales)

| ID  | Prueba                           | Condición                                       | Métrica esperada                                                                 |
| --- | -------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------------- |
| PF-01 | Tiempo de fusión                 | Imágenes de 1024x1024, conexión 4G/WiFi         | < 15 segundos (mayoría de casos < 10 s).                                         |
| PF-02 | Consumo de memoria               | Durante la fusión, mantener un bitmap de 1024x1024 en memoria. | Pico de memoria < 256 MB (evitar `OutOfMemoryError`).                            |
| PF-03 | Tamaño de la imagen guardada     | Tras guardar, comprobar tamaño del archivo.     | Entre 200 KB y 2 MB (depende del contenido).                                     |
| PF-04 | Arranque de la app               | Desde frío (app no en segundo plano).           | Tiempo de primer pantalla < 2 segundos.                                          |

---

## 6. Criterios de aceptación

Para considerar el prototipo **funcional y listo para pruebas de usuario**, se deben cumplir:

- **100% de las pruebas unitarias (UT-01 a UT-12)** pasan en entorno de desarrollo.
- **80% de las pruebas de integración (IT-01 a IT-05)** pasan con mocks; IT-06 (API real) al menos una vez con éxito.
- **Todas las pruebas de UI manuales (UI-01 a UI-13)** pasan en un dispositivo de referencia (Pixel 6 con Android 14).
- **Las pruebas de rendimiento (PF-01 a PF-04)** están dentro de los límites aceptables.

---

## 7. Herramientas sugeridas

| Tipo de prueba        | Herramienta                                 |
| --------------------- | ------------------------------------------- |
| Unitarias             | JUnit 4/5 + MockK (o Mockito)               |
| Integración (ViewModel) | `runTest` + `TestDispatcher` de Corutinas |
| UI automática (opcional) | Espresso + Compose Test                    |
| UI manual             | Lista de verificación (checklist)           |
| Rendimiento           | Android Studio Profiler                     |
