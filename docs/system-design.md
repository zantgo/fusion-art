# Diseño del sistema – FusionArt IA (Prototipo)

## 1. Visión general

El sistema FusionArt IA es una aplicación móvil Android que se comunica con un servicio externo de IA (Google Gemini) para fusionar dos imágenes. El diseño sigue una arquitectura de **cliente-servidor** simple, donde el dispositivo actúa como cliente y la API de IA como servidor.

No hay backend propio ni base de datos centralizada. Toda la lógica de negocio reside en la aplicación móvil.

---

## 2. Diagrama de despliegue (textual)

```
┌───────────────────────────┐
│      Dispositivo Android   │
│  ┌───────────────────────┐ │
│  │   FusionArt IA App    │ │
│  │  (Kotlin + Compose)   │ │
│  └───────────┬───────────┘ │
│              │              │
│  ┌───────────▼───────────┐ │
│  │    Almacenamiento     │ │
│  │    local (MediaStore) │ │
│  └───────────────────────┘ │
└───────────────┬─────────────┘
                │
                │ HTTPS (API calls)
                ▼
┌─────────────────────────────────┐
│      Internet                   │
└───────────────┬─────────────────┘
                │
                ▼
┌─────────────────────────────────┐
│   Google Gemini API             │
│   (generativelanguage.googleapis.com) │
└─────────────────────────────────┘
```

No hay componentes adicionales (balanceadores, colas, etc.) para mantener la simplicidad del prototipo.

---

## 3. Componentes internos de la aplicación

| Componente         | Responsabilidad                                      | Tecnología               |
|--------------------|------------------------------------------------------|--------------------------|
| **ImagePicker**    | Lanzar el selector de galería y obtener URI         | `ActivityResultContracts`|
| **ImageResizer**   | Redimensionar bitmap a tamaño máximo (1024x1024)    | `Bitmap.createScaledBitmap` |
| **ApiClient**      | Configurar Retrofit y realizar la petición HTTP     | Retrofit + OkHttp        |
| **FusionRepository**| Orquestar redimensionado, llamada API, decodificado | Kotlin interface         |
| **FusionViewModel**| Manejar estado UI y coordinar casos de uso          | Jetpack ViewModel + StateFlow |
| **MainScreen**     | UI principal (selección de imágenes, botón fusionar)| Jetpack Compose          |
| **ResultScreen**   | UI para mostrar resultado y guardar                 | Jetpack Compose          |
| **MediaSaver**     | Guardar Bitmap en galería usando MediaStore         | `MediaStore.Images`      |

---

## 4. Flujo de datos (secuencia)

### 4.1 Selección de imágenes

```
Usuario → UI (ImagePicker) → Sistema (galería) → URI → ViewModel → Estado actualizado → UI muestra miniatura
```

### 4.2 Fusión exitosa

```
Usuario (pulsa "Fusionar")
    ↓
ViewModel (onFusionRequested)
    ↓
UseCase (FusionImageUseCase.invoke)
    ↓
Repository.fuse(backgroundUri, foregroundUri)
    ↓ (redimensiona ambas imágenes)
    ↓ (convierte a Base64)
    ↓ (llama a ApiClient)
    ↓
Google Gemini API
    ↓ (devuelve Base64 de imagen fusionada)
    ↓
Repository decodifica a Bitmap
    ↓
UseCase devuelve Result.Success(bitmap)
    ↓
ViewModel actualiza estado → FusionState.Success(bitmap)
    ↓
UI (ResultScreen) muestra la imagen
```

### 4.3 Guardado en galería

```
Usuario (pulsa "Guardar")
    ↓
ViewModel (onSaveRequested)
    ↓
SaveImageUseCase(bitmap)
    ↓
MediaSaver.saveToGallery(bitmap)
    ↓
MediaStore inserta imagen
    ↓
Resultado (URI de la imagen guardada)
    ↓
ViewModel → UI muestra mensaje "Guardado correctamente"
```

---

## 5. Decisiones de diseño específicas

### 5.1 Formato de imágenes en la API
- **Entrada**: Las imágenes se codifican en **Base64** dentro del JSON de la petición (según lo soporta Gemini). Alternativamente, se podría usar `multipart/form-data`, pero Base64 simplifica el cliente.
- **Salida**: La API devuelve la imagen fusionada también en Base64 dentro del JSON.

### 5.2 Redimensionamiento
- Se aplica un límite de **1024x1024 píxeles** (manteniendo la relación de aspecto) antes de enviar a la API.
- Motivo: reducir tiempo de subida, evitar costes elevados (muchas APIs cobran por píxel) y prevenir `OutOfMemoryError`.

### 5.3 Timeout
- **15 segundos** para la llamada a la API. Si excede, se lanza excepción de timeout y se muestra error al usuario.

### 5.4 Manejo de errores
- **Sin conexión**: Se detecta mediante `NetworkCallback` o capturando excepción de conectividad.
- **Error de API (4xx, 5xx)**: Se extrae mensaje del cuerpo de error y se muestra.
- **Formato inválido**: Si la imagen devuelta no es decodificable, se notifica error.

---

## 6. Consideraciones de rendimiento

| Aspecto               | Medida tomada                                           |
|-----------------------|---------------------------------------------------------|
| Memoria               | Los Bitmaps se reciclan adecuadamente; se evita mantener múltiples copias. |
| Red                   | Las imágenes se comprimen a JPEG calidad 85% antes de Base64. |
| UI                   | Las operaciones pesadas (red, redimensionado) se ejecutan en `Dispatchers.IO`. |
| Almacenamiento        | No se guardan imágenes temporales; todo se mantiene en memoria durante el flujo. |

---

## 7. Diagrama de estados del ViewModel

```
[Idle] ──(seleccionar fondo)──> [BackgroundSelected]
                                          │
                     (seleccionar primer plano)
                                          ▼
                              [BothSelected] ──(pulsar Fusionar)──> [Loading]
                                                                         │
                                                        (éxito)         │ (error)
                                                            ▼            ▼
                                                    [Success]       [Error]
                                                         │              │
                                            (Nueva fusión)│  (Reintentar)│
                                                         ▼              ▼
                                                      [Idle] <──────────┘
```

---

## 8. Seguridad (a nivel de sistema)

- **API Key**: No se envía en el código fuente. Se inyecta mediante `BuildConfig` desde `local.properties`.
- **Comunicación**: Todas las peticiones van por HTTPS (TLS 1.2+).
- **Permisos**: La app solo solicita acceso a la galería (`READ_MEDIA_IMAGES`, `WRITE_EXTERNAL_STORAGE` en Android <10). En Android 10+ se usa `MediaStore` sin permiso de escritura para guardar.

---

## 9. Limitaciones del diseño actual (prototipo)

- No soporta imágenes desde la cámara directamente.
- No permite cancelar la fusión una vez iniciada.
- No hay cola de peticiones; si el usuario pulsa múltiples veces "Fusionar", se ignoran mientras `Loading`.
- No se persisten las imágenes seleccionadas al rotar la pantalla (se pierden). Se soluciona con `rememberSaveable` en Compose o guardando URIs en el ViewModel con `SavedStateHandle`.

---

## 10. Evolución futura (diseño escalable)

Si se convierte en producto final, se recomienda:

- Añadir un **backend propio** que gestione las claves de API y cachee resultados.
- Implementar **descarga en segundo plano** con WorkManager.
- Soporte para **múltiples proveedores de IA** mediante un adaptador.
- **Base de datos local** (Room) para historial.
- **Autenticación de usuarios** (Firebase Auth).
