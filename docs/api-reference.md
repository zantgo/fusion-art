# Referencia de la API – FusionArt IA (Prototipo)

## 1. API seleccionada

Para el prototipo se utiliza **Google Gemini 2.5 Flash** a través de su endpoint de generación de contenido con modalidad de imagen. Esta API permite enviar imágenes en Base64 junto con un prompt de texto y recibir una imagen generada o transformada.

**Base URL:**  
`https://generativelanguage.googleapis.com/v1beta`

**Modelo:** `gemini-2.5-flash`

---

## 2. Endpoint para fusión

### 2.1. Generar contenido con imágenes

```
POST /models/gemini-2.5-flash:generateContent?key={API_KEY}
```

**Autenticación:**  
API Key vía query parameter `key` o header `X-Goog-Api-Key`. Para simplificar el prototipo se usará el parámetro `key`.

**Content-Type:** `application/json`

---

## 3. Formato de la petición (Request)

### 3.1. Estructura JSON

```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "Fusiona la imagen de fondo (primera) con el objeto de la imagen de primer plano (segunda). Ajusta iluminación, sombras y perspectiva para que parezca realista. Devuelve solo la imagen fusionada."
        },
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "base64_encoded_background_image"
          }
        },
        {
          "inlineData": {
            "mimeType": "image/jpeg",
            "data": "base64_encoded_foreground_image"
          }
        }
      ]
    }
  ],
  "generationConfig": {
    "responseModalities": ["IMAGE"],
    "temperature": 0.4
  }
}
```

### 3.2. Campos de la petición

| Campo                                     | Tipo   | Obligatorio | Descripción                                                                 |
| ----------------------------------------- | ------ | ----------- | --------------------------------------------------------------------------- |
| `contents[0].parts[0].text`               | string | Sí          | Prompt que describe la fusión deseada.                                      |
| `contents[0].parts[1].inlineData`         | object | Sí          | Primera imagen (fondo).                                                     |
| `contents[0].parts[2].inlineData`         | object | Sí          | Segunda imagen (primer plano).                                              |
| `inlineData.mimeType`                     | string | Sí          | Tipo MIME de la imagen. Valores soportados: `image/jpeg`, `image/png`, `image/webp`. |
| `inlineData.data`                         | string | Sí          | Imagen codificada en Base64 (sin el prefijo `data:image/...;base64,`).       |
| `generationConfig.responseModalities`     | array  | No          | Obligatorio para que devuelva imagen. Valor fijo: `["IMAGE"]`.               |
| `generationConfig.temperature`            | number | No          | Controla la creatividad (0.0 a 1.0). Para fusión se recomienda 0.4.         |

> **Nota:** El prompt puede ajustarse según pruebas. El mostrado es el recomendado para fusión coherente.

### 3.3. Limitaciones de la imagen (aplicadas en cliente)

- Tamaño máximo antes de codificar: **1024x1024 píxeles** (redimensionado automático).
- Tamaño máximo del archivo después de Base64: aproximadamente **4 MB** (Gemini acepta hasta 20 MB, pero se limita por rendimiento).
- Formatos aceptados: JPEG, PNG, WEBP.

---

## 4. Formato de la respuesta (Response)

### 4.1. Respuesta exitosa (HTTP 200)

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "inlineData": {
              "mimeType": "image/png",
              "data": "base64_encoded_fused_image"
            }
          }
        ]
      },
      "finishReason": "STOP",
      "index": 0
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 1234,
    "candidatesTokenCount": 567,
    "totalTokenCount": 1801
  }
}
```

### 4.2. Campos de la respuesta

| Campo                                            | Descripción                                                      |
| ------------------------------------------------ | ---------------------------------------------------------------- |
| `candidates[0].content.parts[0].inlineData.data` | Imagen fusionada en Base64 (sin prefijo).                        |
| `candidates[0].finishReason`                     | Razón de finalización. Para éxito: `"STOP"`.                     |
| `usageMetadata.promptTokenCount`                 | Cantidad de tokens utilizados en la entrada (texto + imágenes).  |
| `usageMetadata.candidatesTokenCount`             | Tokens de salida (imagen).                                       |
| `usageMetadata.totalTokenCount`                  | Suma de los anteriores.                                          |

### 4.3. Decodificación de la imagen

La aplicación debe:
1. Extraer el string Base64 de `candidates[0].content.parts[0].inlineData.data`.
2. Decodificarlo a `ByteArray`.
3. Convertir el `ByteArray` a `Bitmap` mediante `BitmapFactory.decodeByteArray()`.

---

## 5. Códigos de error HTTP

| Código | Significado                           | Acción en la app                                 |
| ------ | ------------------------------------- | ------------------------------------------------ |
| 200    | OK                                    | Procesar imagen normalmente.                     |
| 400    | Bad Request                           | Mostrar: "Formato de imagen no soportado o prompt inválido". |
| 401    | Unauthorized                          | Mostrar: "Clave de API inválida. Verifica la configuración". |
| 403    | Forbidden                             | Mostrar: "Acceso denegado. La clave no tiene permisos para este modelo". |
| 429    | Too Many Requests                     | Mostrar: "Demasiadas solicitudes. Espera unos segundos". |
| 500    | Internal Server Error                 | Mostrar: "Error en el servidor de IA. Intenta más tarde". |
| 503    | Service Unavailable                   | Mostrar: "Servicio temporalmente no disponible". |
| 504    | Gateway Timeout                       | Mostrar: "Tiempo de espera agotado. La IA no respondió". |

---

## 6. Ejemplo completo de llamada (usando Retrofit)

### 6.1. Definición del servicio en Kotlin

```kotlin
interface GeminiApiService {
    @POST("models/gemini-2.5-flash:generateContent")
    @Headers("Content-Type: application/json")
    suspend fun generateContent(
        @Query("key") apiKey: String,
        @Body request: GeminiRequest
    ): Response<GeminiResponse>
}
```

### 6.2. Clases de datos (DTOs)

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
    val data: String  // Base64
)

data class GenerationConfig(
    val responseModalities: List<String>,
    val temperature: Double
)

data class GeminiResponse(
    val candidates: List<Candidate>
)

data class Candidate(
    val content: ContentResponse
)

data class ContentResponse(
    val parts: List<PartResponse>
)

data class PartResponse(
    val inlineData: InlineDataResponse?
)

data class InlineDataResponse(
    val mimeType: String,
    val data: String
)
```

### 6.3. Construcción de la petición (desde URIs)

```kotlin
suspend fun createRequest(backgroundUri: Uri, foregroundUri: Uri): GeminiRequest {
    val backgroundBase64 = uriToBase64(backgroundUri)
    val foregroundBase64 = uriToBase64(foregroundUri)
    
    return GeminiRequest(
        contents = listOf(
            Content(
                parts = listOf(
                    Part(text = "Fusiona la imagen de fondo (primera) con el objeto de la imagen de primer plano (segunda). Ajusta iluminación, sombras y perspectiva para que parezca realista. Devuelve solo la imagen fusionada."),
                    Part(inlineData = InlineData("image/jpeg", backgroundBase64)),
                    Part(inlineData = InlineData("image/jpeg", foregroundBase64))
                )
            )
        ),
        generationConfig = GenerationConfig(
            responseModalities = listOf("IMAGE"),
            temperature = 0.4
        )
    )
}
```

### 6.4. Función auxiliar para convertir URI a Base64

```kotlin
private suspend fun uriToBase64(uri: Uri): String = withContext(Dispatchers.IO) {
    val inputStream = context.contentResolver.openInputStream(uri)
    val bytes = inputStream?.readBytes() ?: throw IllegalArgumentException("No se pudo leer la imagen")
    val resized = resizeBitmap(bytes, maxSize = 1024)
    val outputStream = ByteArrayOutputStream()
    resized.compress(Bitmap.CompressFormat.JPEG, 85, outputStream)
    Base64.encodeToString(outputStream.toByteArray(), Base64.NO_WRAP)
}
```

---

## 7. API alternativa (Adobe Firefly) – para referencia futura

Si se decide cambiar de proveedor, el endpoint sería:

```
POST https://firefly-api.adobe.io/v2/images/composite
Headers:
  x-api-key: {API_KEY}
  Authorization: Bearer {ACCESS_TOKEN}
  Content-Type: multipart/form-data
Body:
  - background: archivo
  - foreground: archivo
  - options: '{"shadow": "auto", "scale": "fit", "position": "center"}'
```

No se detalla completamente porque el prototipo usa Gemini, pero la arquitectura permite intercambiar implementaciones.

---

## 8. Consideraciones de seguridad

- La **API Key** nunca debe enviarse en código fuente. Se inyecta mediante `BuildConfig.GEMINI_API_KEY`.
- Todas las comunicaciones son HTTPS (TLS 1.2+).
- No se almacenan las imágenes enviadas ni recibidas en disco (solo en memoria durante el flujo).
