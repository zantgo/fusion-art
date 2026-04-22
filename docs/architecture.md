Claro, aquí tienes el archivo `architecture.md` completo, pensado para un prototipo sencillo pero bien estructurado.

---

```markdown
# Arquitectura del software – FusionArt IA (Prototipo)

## 1. Enfoque general

Para este prototipo se utiliza una **arquitectura limpia simplificada (Clean Architecture)** que separa la aplicación en tres capas principales:

- **Presentación** (UI + ViewModels)
- **Dominio** (casos de uso y modelos de negocio)
- **Datos** (repositorios, fuentes de datos locales y remotas)

Esta separación permite:
- Mantener el código desacoplado y testeable.
- Cambiar fácilmente la API de IA (Gemini, Firefly, etc.) sin afectar la UI.
- Evolucionar el prototipo hacia una versión completa sin reescribir todo.

---

## 2. Diagrama de capas (textual)

```
┌─────────────────────────────────────────────────────┐
│                 Capa de Presentación                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │   Screens   │  │  ViewModels │  │   Estados   │  │
│  │ (Compose)   │◄─┤ (StateFlow) │  │ (sealed)    │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│                   Capa de Dominio                    │
│  ┌─────────────────┐  ┌──────────────────────────┐  │
│  │  Casos de uso   │  │     Modelos de negocio   │  │
│  │ (FusionImage,   │  │ (ImageData, FusionResult)│  │
│  │  SaveImage)     │  │                          │  │
│  └─────────────────┘  └──────────────────────────┘  │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│                    Capa de Datos                     │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ Repositorio  │  │ Fuente local │  │ Fuente remota│ │
│  │ (FusionRepo) │  │ (MediaStore) │  │ (API Gemini) │ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## 3. Tecnologías y bibliotecas

| Capa / Función       | Tecnología elegida                     | Motivo                                       |
|----------------------|----------------------------------------|----------------------------------------------|
| Lenguaje             | Kotlin                                 | Estándar en Android, moderno y seguro       |
| UI                   | Jetpack Compose + Material 3           | Menos código boilerplate, reactivo           |
| Inyección de dependencias | Manual (por constructor)           | Simplicidad para el prototipo                |
| Comunicación con API | Retrofit + OkHttp + Moshi (JSON)       | Ampliamente usado, fácil configuración       |
| Imágenes             | Coil (carga, redimensionamiento)       | Ligero, compatible con Compose               |
| Almacenamiento local | MediaStore API (para guardar imagen)   | API nativa, sin dependencias extra           |
| Permisos             | Accompanist Permissions (o nativo)     | Simplifica la gestión en tiempo de ejecución |
| Asincronía           | Kotlin Coroutines + Flow               | Ligero, integración nativa con ViewModel     |
| Estado en UI         | StateFlow / MutableStateFlow           | Reactivo y fácil de probar                   |

---

## 4. Descripción de cada capa

### 4.1 Capa de presentación
- **Screens**: Composables que muestran la UI y lanzan eventos.
- **ViewModels**: Almacenan el estado (`FusionState`) y procesan las acciones del usuario (seleccionar imagen, fusionar, guardar).
- **Eventos**: Se definen como funciones en el ViewModel (ej. `onBackgroundSelected(uri)`).

### 4.2 Capa de dominio
- **Casos de uso**: Contienen la lógica de negocio pura.
  - `FusionImageUseCase`: Recibe dos imágenes (Uri o Bitmap) y orquesta la llamada al repositorio.
  - `SaveImageUseCase`: Guarda el Bitmap resultante en la galería.
- **Modelos**: `ImageData`, `FusionResult` (sealed class para éxito/error).

### 4.3 Capa de datos
- **Repositorio**: Implementa una interfaz definida en dominio. Ejemplo:
  ```kotlin
  interface FusionRepository {
      suspend fun fuse(background: Uri, foreground: Uri): Result<Bitmap>
  }
  ```
- **Fuentes remotas**: Cliente Retrofit para llamar a la API de Gemini.
- **Fuentes locales**: Funciones para redimensionar imágenes y guardar en MediaStore.
- **Mapeadores**: Convierten entre DTOs de red y modelos de dominio.

---

## 5. Flujo de datos típico (fusión de imágenes)

1. **Usuario** selecciona fondo y primer plano → UI envía URIs al ViewModel.
2. **ViewModel** llama a `FusionImageUseCase.invoke(backgroundUri, foregroundUri)`.
3. **UseCase** invoca al repositorio (`fusionRepository.fuse(...)`).
4. **Repositorio**:
   - Redimensiona las imágenes (utilizando un helper en `utils/`).
   - Convierte URIs a Base64 (o multipart).
   - Llama a la API de IA mediante Retrofit.
   - Recibe la respuesta (Base64 de la imagen fusionada).
   - Decodifica a Bitmap.
5. **UseCase** devuelve el Bitmap al ViewModel (envuelto en `Result.Success` o `Result.Error`).
6. **ViewModel** actualiza el estado (`FusionState.Success(bitmap)`).
7. **UI** (Compose) recompondrá la pantalla mostrando el resultado.

---

## 6. Gestión de errores

- Los errores de red, timeout o respuestas 4xx/5xx se capturan en el repositorio y se propagan como `Result.Error(mensaje)`.
- El ViewModel traduce esos mensajes a strings legibles por el usuario.
- En la UI se muestra un `Snackbar` o un diálogo con el error.

---

## 7. Decisiones clave para el prototipo

| Decisión | Alternativa descartada | Motivo |
|----------|------------------------|--------|
| No usar Dagger/Hilt | Koin o Hilt | Simplifica la curva de aprendizaje; la inyección manual es suficiente para ~5 clases. |
| API Gemini como primera opción | Adobe Firefly | Gemini tiene un modelo gratuito y documentación clara para generación de imágenes. |
| Redimensionar imágenes a 1024x1024 | Enviar tamaño original | Reduce tiempo de subida, costo de API y evita `OutOfMemoryError`. |
| No usar base de datos local | Room | El prototipo no necesita persistir fusiones previas. |
| UI con Compose | XML + ViewBinding | Menos código, más rápido de iterar. |

---

## 8. Seguridad (simple)

- La API Key se guarda en `local.properties` y se inyecta en tiempo de compilación mediante `BuildConfig`.
- En producción, se recomienda usar un backend proxy o Firebase Remote Config, pero para el prototipo local es aceptable.

---

## 9. Extensibilidad para el futuro

Si el prototipo evoluciona a versión completa, se pueden añadir:

- **Inyección de dependencias** con Hilt.
- **Base de datos** Room para historial de fusiones.
- **Capa de cache** de imágenes con Coil.
- **Soporte para múltiples modelos de IA** mediante patrón Strategy.
- **Pruebas unitarias** completas con MockK.

---

## 10. Resumen visual (simplificado)

```
[UI] → [ViewModel] → [UseCase] → [Repository] → [API Gemini]
  ↑         │            │            │
  └─────────┴────────────┴────────────┘
        (Estados y eventos)
```

---

> **Nota:** Esta arquitectura es intencionalmente simple para que el prototipo sea funcional en pocos días. No se considera sobreingeniería.
```

---

Este archivo `architecture.md` está listo para ser copiado en la carpeta `docs/` del proyecto. ¿Quieres que continúe con el siguiente (`system-design.md`) o prefieres ajustar algo aquí?