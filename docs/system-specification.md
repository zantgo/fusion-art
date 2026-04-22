# Especificación del Sistema: FusionArt IA

## 1. Propósito y Alcance

**Propósito:**  
Desarrollar un prototipo funcional de aplicación móvil Android que permita a un usuario seleccionar dos imágenes (fondo y primer plano) y, mediante una llamada a una API de inteligencia artificial, generar una nueva imagen que fusione coherentemente ambas, respetando iluminación, perspectiva y contexto.

**Alcance del prototipo:**  
- Selección de imágenes desde la galería del dispositivo.  
- Previsualización de las imágenes seleccionadas.  
- Invocación a una API de I
- Visualización del resultado fusionado.  
- Opción para guardar la imagen resultante en la galería.  
- Manejo básico de errores (red, formato, tamaño).  

**Fuera de alcance (para este prototipo):**  
- Edición manual de la fusión (recortes, ajustes de color).  
- Procesamiento completamente offline (siempre requiere conexión a internet).  
- Soporte para video o lotes de imágenes.  
- Autenticación de usuarios o almacenamiento en la nube.

---

## 2. Requisitos Funcionales (RF)

| ID   | Descripción |
|------|-------------|
| RF01 | La aplicación debe permitir seleccionar una imagen de fondo desde la galería del dispositivo (formatos JPEG, PNG, WEBP). |
| RF02 | La aplicación debe permitir seleccionar una imagen de primer plano desde la galería del dispositivo (mismos formatos). |
| RF03 | Tras seleccionar ambas imágenes, se mostrarán en miniatura para confirmación visual. |
| RF04 | Debe existir un botón "Fusionar" que, al presionarse, inicie el proceso de llamada a la API de IA. |
| RF05 | Durante la llamada a la API, se debe mostrar un indicador de progreso (loader) y deshabilitar el botón para evitar múltiples envíos. |
| RF06 | La aplicación debe enviar ambas imágenes a la API en formato multipart/form-data o Base64, según requiera el servicio. |
| RF07 | La API devolverá la imagen fusionada en formato JPEG o PNG. La aplicación la descargará y mostrará en pantalla. |
| RF08 | La imagen resultante se podrá guardar en la galería del dispositivo mediante un botón "Guardar". |
| RF09 | La aplicación debe solicitar los permisos necesarios en tiempo de ejecución (READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE, o las variantes de Android 10+ como ACCESS_MEDIA_LOCATION). |
| RF10 | En caso de error (red, timeout, API no responde, formato inválido), se debe mostrar un mensaje claro al usuario. |
| RF11 | Se debe permitir reiniciar el flujo (cambiar de imágenes) sin salir de la aplicación. |

---

## 3. Requisitos No Funcionales (RNF)

| ID    | Descripción |
|-------|-------------|
| RNF01 | **Rendimiento:** La llamada a la API no debe exceder los 15 segundos (tiempo de espera típico). La interfaz debe mantenerse responsiva. |
| RNF02 | **Usabilidad:** La interfaz será intuitiva, con botones grandes y texto claro. Soporte para modo oscuro/claro del sistema. |
| RNF03 | **Seguridad:** Las claves de API no se almacenarán en código fuente; se usarán variables de entorno o un servicio de configuración remota (o al menos ofuscadas con BuildConfig). |
| RNF04 | **Compatibilidad:** Android mínimo API 24 (Android 7.0). Probado en versiones hasta API 35 (Android 15). |
| RNF05 | **Tamaño de imagen:** La aplicación redimensionará las imágenes a un máximo de 2048x2048 píxeles antes de enviarlas para reducir uso de datos y tiempo de procesamiento. |
| RNF06 | **Consumo de batería:** No debe haber procesos en segundo plano persistentes. El consumo se limita al momento de la llamada API. |
| RNF07 | **Idioma:** Interfaz en español (por defecto) con soporte para inglés (strings.xml). |

---

## 4. Arquitectura y Tecnologías

### 4.1. Diagrama de componentes

```
[Android App (Kotlin + Jetpack Compose)]
          |
          | (Retrofit/OkHttp)
          v
      [API Gateway]
          |
          | (HTTPS)
          v
      [Servicio de IA
          |
          | (Imagen fusionada)
          v
      [Almacenamiento temporal en memoria]
          |
          | (Guardado)
          v
   [Galería del dispositivo]
```

### 4.2. Stack tecnológico

- **Lenguaje:** Kotlin 1.9+  
- **UI:** Jetpack Compose (Material 3)  
- **Networking:** Retrofit 2.9 + OkHttp 4.12  
- **Manejo de imágenes:** Coil 2.5 (para carga y redimensionamiento)  
- **Permisos:** Accompanist Permissions (o API nativa)  
- **Guardado en galería:** `MediaStore` API  
- **Pruebas (opcional):** JUnit, Espresso (para UI)  

### 4.3. Estructura de paquetes (Clean Architecture simplificada)

```
com.fusionart.ia/
├── data/
│   ├── api/          (Retrofit interfaces, modelos de request/response)
│   ├── repository/   (Implementación de repositorio)
│   └── datasource/   (Local: MediaStore, Remote: API)
├── domain/
│   ├── model/        (ImageData, FusionResult)
│   └── usecase/      (FusionImageUseCase, SaveImageUseCase)
├── presentation/
│   ├── screen/       (Composables: MainScreen, ResultScreen)
│   ├── viewmodel/    (FusionViewModel - StateFlow)
│   └── components/   (Botones, Loaders, ImagePickers)
└── utils/            (ImageResizer, PermissionHelper, ApiKeyHelper)
```

---

## 5. Diseño de Interfaz de Usuario (UI/UX)

### 5.1. Flujo de pantallas

1. **Pantalla principal (MainScreen)**  
   - Dos tarjetas: "Fondo" y "Primer plano".  
   - Cada tarjeta tiene un ícono de cámara/galería y muestra la imagen seleccionada en miniatura.  
   - Botón "Fusionar" (inicialmente deshabilitado hasta tener ambas imágenes).  
   - Opción "Limpiar todo" (reset).  

2. **Pantalla de carga (modal o pantalla completa)**  
   - Indicador circular.  
   - Mensaje: "Fusionando imágenes con IA...".  

3. **Pantalla de resultado (ResultScreen)**  
   - Imagen fusionada a tamaño completo (ajustada al ancho).  
   - Botón "Guardar en galería".  
   - Botón "Nueva fusión" (vuelve a pantalla principal).  

### 5.2. Wireframe conceptual (textual)

```
-----------------------------------
|           FusionArt IA           |
-----------------------------------
|  [Fondo]              [Primer plano] |
|  + Seleccionar         + Seleccionar |
|  (vista previa)        (vista previa)|
|                                       |
|          [    FUSIONAR    ]          |
|                                       |
|          [ Limpiar todo ]             |
-----------------------------------
```

### 5.3. Paleta de colores (Material 3)

- Primary: #6750A4 (púrpura)  
- Secondary: #625B71  
- Surface: #FFFBFE  
- Error: #BA1A1A  
- Tipografía: Roboto  

---

## 6. Integración con API de IA (Detalle Técnico)

### 6.1. API primaria seleccionada: Google Gemini 2.5 Flash (modalidad imagen a imagen)

**Endpoint:** `POST https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key={API_KEY}`  

**Formato de petición (ejemplo simplificado):**  
```json
{
  "contents": [
    {
      "parts": [
        { "text": "Fusiona la imagen de fondo (primera) con el objeto de la imagen de primer plano (segunda). Ajusta iluminación, sombras y perspectiva para que parezca realista. Devuelve solo la imagen fusionada." },
        { "inlineData": { "mimeType": "image/jpeg", "data": "base64_fondo" } },
        { "inlineData": { "mimeType": "image/jpeg", "data": "base64_primero" } }
      ]
    }
  ],
  "generationConfig": {
    "responseModalities": ["IMAGE"]
  }
}
```

**Respuesta esperada:**  
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "inlineData": {
              "mimeType": "image/png",
              "data": "base64_imagen_fusionada"
            }
          }
        ]
      }
    }
  ]
}
```

### 6.2. API alternativa (Adobe Firefly - Composite)

**Endpoint:** `POST https://firefly-api.adobe.io/v2/images/composite`  
**Headers:** `x-api-key`, `Authorization: Bearer <token>`  
**Body (multipart):**  
- `background` (archivo)  
- `foreground` (archivo)  
- `options` (JSON con ajustes de sombra, escala, posición automática)

### 6.3. Manejo de errores de API

| Código HTTP | Significado | Acción en app |
|-------------|-------------|----------------|
| 200 | OK | Procesar imagen |
| 400 | Solicitud incorrecta (imagen no válida) | Mostrar "Formato de imagen no soportado" |
| 401/403 | Clave inválida o cuota excedida | Mostrar "Error de autenticación. Contacte soporte" |
| 429 | Demasiadas solicitudes | Mostrar "Espera unos segundos e intenta de nuevo" |
| 5xx | Error del servidor | Mostrar "Error temporal. Intenta más tarde" |
| Timeout | >15 segundos sin respuesta | Mostrar "Tiempo de espera agotado" |

---

## 7. Modelo de Datos (Entidades locales)

### 7.1. `ImageData` (sealed class / data class)

```kotlin
sealed class ImageData {
    object Empty : ImageData()
    data class Selected(val bitmap: Bitmap, val uri: Uri, val mimeType: String) : ImageData()
}
```

### 7.2. `FusionState` (para ViewModel)

```kotlin
sealed class FusionState {
    object Idle : FusionState()
    object Loading : FusionState()
    data class Success(val imageBitmap: Bitmap) : FusionState()
    data class Error(val message: String) : FusionState()
}
```

### 7.3. Preferencias

- Almacenar la última API seleccionada (Gemini o Firefly) en `DataStore` (para futura expansión).  
- No se guardan las imágenes seleccionadas entre sesiones.

---

## 8. Requisitos de Seguridad y Privacidad

- **API Key:** No se hardcodea; se leerá desde `local.properties` (para desarrollo) y en producción mediante servicio seguro (Firebase Remote Config o secreto en backend propio).  
- **Permisos:** Solicitar solo los necesarios en tiempo de ejecución y explicar el motivo.  
- **Datos del usuario:** Las imágenes subidas a la API no se almacenan en servidores propios; la API de terceros puede retenerlas según sus políticas (se debe informar al usuario en un aviso legal).  
- **Comunicación:** Todo tráfico va por HTTPS.

---

## 9. Plan de Pruebas (Estrategia para prototipo)

| Tipo de prueba | Herramienta / Método | Cobertura |
|----------------|----------------------|------------|
| Unitarias | JUnit + MockK (para Repository y UseCase) | Lógica de negocio, redimensionamiento, validaciones |
| Integración | Pruebas manuales con API real (Gemini) | Flujo completo de fusión |
| UI | Pruebas manuales en dispositivos físicos (Android 10, 13, 14) | Navegación, selección de imágenes, guardado |
| Rendimiento | Perfilador de Android Studio | Consumo de memoria al manejar bitmaps grandes |
| Seguridad | Inspección estática (no hardcodeo de claves) | Verificar ausencia de secretos en el repo |

---

## 10. Riesgos y Mitigación

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|-------------|
| API de IA no devuelve fusión coherente | Media | Alto | Probar con múltiples prompts y tener plan B (Adobe Firefly). Incluir slider de ajuste manual en futura versión. |
| Tamaño de imagen demasiado grande agota memoria | Baja | Medio | Redimensionar a 1024x1024 antes de enviar. Usar `BitmapFactory` con opciones de muestreo. |
| Cambios en la API de Gemini (deprecación) | Baja | Alto | Abstraer el servicio de IA mediante un interface para poder cambiar de proveedor fácilmente. |
| Usuario sin conexión a internet | Media | Bajo | Mostrar mensaje claro: "Se requiere conexión a internet para fusionar". |

---

## 11. Entregables del Desarrollo

- Código fuente completo en repositorio Git (GitHub o GitLab).  
- Documentación de configuración (`README.md`): cómo obtener API Key, cómo compilar y ejecutar.  
- Archivo APK firmado (para pruebas).  
- Capturas de pantalla del flujo funcional.  

---

## 12. Cronograma Estimado (para un desarrollador)

| Fase | Duración |
|------|-----------|
| Configuración del proyecto + permisos + picker de imágenes | 1 día |
| Integración de Retrofit + llamada a Gemini (mock primero) | 1 día |
| Implementación de UI con Compose (pantalla principal y resultado) | 1 día |
| Lógica de redimensionamiento y manejo de errores | 0.5 día |
| Guardado en galería y pruebas de integración | 0.5 día |
| Pruebas en dispositivos, pulido de UI, documentación | 1 día |
| **Total** | **5 días** (40 horas) |

---

## 13. Consideraciones para Futuras Iteraciones (post-protipo)

- Permitir al usuario ajustar posición, escala y rotación del objeto de primer plano.  
- Añadir modo offline con modelo TensorFlow Lite (por ejemplo, DeepLab para segmentación).  
- Soporte para capas múltiples (varios objetos).  
- Historial de fusiones guardadas localmente.  
- Compartir imagen directamente a redes sociales.

---

## 14. Aprobaciones

| Rol | Nombre | Firma | Fecha |
|-----|--------|-------|-------|
| Solicitante (Product Owner) | ________ | _______ | _______ |
| Arquitecto de software | ________ | _______ | _______ |
| Desarrollador líder | ________ | _______ | _______ |
