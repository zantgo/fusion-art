# Modelo de datos – FusionArt IA (Prototipo)

## 1. Introducción

Este documento define las estructuras de datos utilizadas dentro de la aplicación. No se utiliza una base de datos relacional (Room) en el prototipo; los datos se mantienen en memoria (ViewModels, repositorios) o se almacenan temporalmente en URIs y bitmaps. Solo se persiste la imagen final en la galería mediante MediaStore.

---

## 2. Entidades principales

| Entidad            | Descripción                                                                 |
| ------------------ | --------------------------------------------------------------------------- |
| `ImageData`        | Representa una imagen seleccionada (fondo o primer plano).                  |
| `FusionState`      | Estado de la UI en el ViewModel (idle, loading, success, error).            |
| `FusionResult`     | Resultado de la operación de fusión (éxito con Bitmap, error con mensaje).  |
| `ImageResizeConfig`| Configuración para el redimensionado de imágenes antes de enviar a la API.  |
| `ApiRequest`       | DTO para la petición a la API de Gemini (conversión a JSON).                |
| `ApiResponse`      | DTO para la respuesta de la API.                                            |

---

## 3. Descripción detallada de entidades

### 3.1. `ImageData`

```kotlin
sealed class ImageData {
    object Empty : ImageData()
    data class Selected(
        val uri: Uri,
        val bitmapThumbnail: Bitmap?,  // miniatura para previsualización
        val mimeType: String = "image/jpeg"
    ) : ImageData()
}
```

**Campos:**
- `Empty`: Estado sin imagen seleccionada.
- `Selected`: Contiene la URI original (para redimensionado y envío), una miniatura (opcional, para mostrar en UI) y el tipo MIME.

### 3.2. `FusionState` (estado del ViewModel)

```kotlin
sealed class FusionState {
    object Idle : FusionState()
    object Loading : FusionState()
    data class Success(val fusedBitmap: Bitmap) : FusionState()
    data class Error(val message: String) : FusionState()
}
```

**Flujo:**
- `Idle`: Sin imágenes seleccionadas o después de limpiar.
- `Loading`: Mientras se llama a la API.
- `Success`: Fusión completada, contiene el Bitmap resultante.
- `Error`: Ocurrió un error, con mensaje legible para el usuario.

### 3.3. `FusionResult` (resultado del caso de uso)

```kotlin
sealed class FusionResult {
    data class Success(val bitmap: Bitmap) : FusionResult()
    data class Error(val exception: Exception, val userMessage: String) : FusionResult()
}
```

Usado internamente en la capa de dominio/repositorio. El `userMessage` se extrae según el tipo de excepción.

### 3.4. `ImageResizeConfig`

```kotlin
data class ImageResizeConfig(
    val maxWidth: Int = 1024,
    val maxHeight: Int = 1024,
    val quality: Int = 85,      // compresión JPEG (0-100)
    val format: Bitmap.CompressFormat = Bitmap.CompressFormat.JPEG
)
```

Controla cómo se redimensionan las imágenes antes de convertirlas a Base64.

### 3.5. DTOs para API de Gemini

#### Petición (`GeminiRequest`)

```kotlin
data class GeminiRequest(
    val contents: List<Content>,
    val generationConfig: GenerationConfig
)

data class Content(
    val parts: List<Part>
)

data class Part(
    val text: String? = null,
    val inlineData: InlineData? = null
)

data class InlineData(
    val mimeType: String,
    val data: String   // Base64
)

data class GenerationConfig(
    val responseModalities: List<String>,
    val temperature: Double
)
```

#### Respuesta (`GeminiResponse`)

```kotlin
data class GeminiResponse(
    val candidates: List<Candidate>
)

data class Candidate(
    val content: ContentResponse,
    val finishReason: String,
    val index: Int
)

data class ContentResponse(
    val parts: List<PartResponse>
)

data class PartResponse(
    val inlineData: InlineDataResponse?
)

data class InlineDataResponse(
    val mimeType: String,
    val data: String   // Base64 de la imagen fusionada
)
```

---

## 4. Relaciones entre entidades

```
[MainScreen] --> ViewModel --> FusionState
                     |
                     | (usa)
                     v
              FusionImageUseCase
                     |
                     | (retorna)
                     v
              FusionResult (Success/Error)
                     |
                     | (mapea a)
                     v
              FusionState.Success / .Error

[Repository] --> convierte URIs --> GeminiRequest --> GeminiResponse --> Bitmap
```

No hay relaciones de persistencia (tablas). Solo dependencias temporales en memoria.

---

## 5. Preferencias locales (DataStore)

Aunque no es obligatorio para el prototipo, se puede añadir `PreferencesDataStore` para guardar:

- **Último proveedor de IA seleccionado** (Gemini o Firefly) – para futuras versiones.
- **Calidad de imagen por defecto** (ej. 85).

Ejemplo de clave:

```kotlin
object PreferencesKeys {
    val API_PROVIDER = stringPreferencesKey("api_provider")
    val IMAGE_QUALITY = intPreferencesKey("image_quality")
}
```

Pero **no se incluye en el prototipo inicial** para mantener la simplicidad.

---

## 6. Validaciones y restricciones

| Entidad         | Restricción                                                          |
| --------------- | -------------------------------------------------------------------- |
| `ImageData`     | URI no nula, el bitmapThumbnail puede ser nulo si no se carga.      |
| `GeminiRequest` | El texto del prompt no debe estar vacío. Las imágenes en Base64 deben ser menores a 5 MB (aprox). |
| `FusionState.Success` | El Bitmap no debe ser nulo ni estar reciclado.                  |

---

## 7. Ejemplo de uso en el ViewModel

```kotlin
class FusionViewModel : ViewModel() {
    private val _state = MutableStateFlow<FusionState>(FusionState.Idle)
    val state: StateFlow<FusionState> = _state.asStateFlow()

    private var backgroundImage: ImageData = ImageData.Empty
    private var foregroundImage: ImageData = ImageData.Empty

    fun selectBackground(uri: Uri) {
        backgroundImage = ImageData.Selected(uri, loadThumbnail(uri))
        updateFusionButtonState()
    }

    fun fuse() {
        if (backgroundImage is ImageData.Selected && foregroundImage is ImageData.Selected) {
            _state.value = FusionState.Loading
            viewModelScope.launch {
                val result = fusionUseCase.invoke(
                    (backgroundImage as ImageData.Selected).uri,
                    (foregroundImage as ImageData.Selected).uri
                )
                _state.value = when (result) {
                    is FusionResult.Success -> FusionState.Success(result.bitmap)
                    is FusionResult.Error -> FusionState.Error(result.userMessage)
                }
            }
        }
    }
}
```

---

## 8. Diagrama simplificado de clases (textual)

```
┌─────────────────┐     ┌─────────────────┐
│  FusionState    │     │   ImageData     │
├─────────────────┤     ├─────────────────┤
│ +Idle           │     │ +Empty          │
│ +Loading        │     │ +Selected(uri,  │
│ +Success(bitmap)│     │    thumbnail)   │
│ +Error(message) │     └─────────────────┘
└─────────────────┘
         │
         │ (usado por)
         ▼
┌─────────────────────────────────────┐
│         FusionViewModel             │
├─────────────────────────────────────┤
│ - state: StateFlow<FusionState>     │
│ - backgroundImage: ImageData        │
│ - foregroundImage: ImageData        │
│ + selectBackground(uri)             │
│ + selectForeground(uri)             │
│ + fuse()                            │
│ + clear()                           │
└─────────────────────────────────────┘
         │
         │ (usa)
         ▼
┌─────────────────────────────────────┐
│      FusionImageUseCase             │
└─────────────────────────────────────┘
         │
         │ (retorna)
         ▼
┌─────────────────────────────────────┐
│         FusionResult                │
├─────────────────────────────────────┤
│ + Success(bitmap)                   │
│ + Error(exception, userMessage)     │
└─────────────────────────────────────┘
```

---

## 9. Consideraciones de memoria

- Los `Bitmap` de las miniaturas se cargan con Coil, que ya gestiona caché y reciclaje.
- La imagen fusionada (`FusionState.Success.bitmap`) se mantiene en memoria mientras está en ResultScreen. Al salir de esa pantalla, se debe liberar (`bitmap.recycle()`) o dejar que el GC actúe.
- El `ImageResizeConfig` evita crear bitmaps de más de 1024x1024.

---

> **Nota:** Este modelo es suficiente para el prototipo. En una versión completa se añadirían entidades para historial (Room) y ajustes de usuario.