# Esquema de almacenamiento – FusionArt IA (Prototipo)

## 1. Introducción

El prototipo **no utiliza una base de datos SQL (Room)**. Los datos temporales (imágenes seleccionadas, estado de fusión) se mantienen en memoria (ViewModel). La única persistencia ocurre cuando el usuario guarda la imagen fusionada, que se almacena en la galería del dispositivo utilizando la API de **MediaStore**. Además, se reserva un espacio para **Preferencias (DataStore)** que podrían usarse en futuras iteraciones.

---

## 2. Almacenamiento de imágenes: MediaStore

### 2.1. Ubicación física

| Sistema operativo | Ruta aproximada                                      |
| ----------------- | ---------------------------------------------------- |
| Android 10+       | `/storage/emulated/0/Pictures/FusionArtIA/`        |
| Android 9 y anteriores | `/storage/emulated/0/Pictures/FusionArtIA/`   (misma ruta, pero con permisos diferentes) |

La carpeta `FusionArtIA` se crea automáticamente al guardar la primera imagen.

### 2.2. Esquema de la tabla `MediaStore.Images`

No se crea una tabla personalizada, sino que se inserta una fila en el proveedor de contenido `MediaStore.Images.Media`. Las columnas relevantes son:

| Columna                        | Tipo      | Valor que asigna la app                                        |
| ------------------------------ | --------- | -------------------------------------------------------------- |
| `DISPLAY_NAME`                 | String    | `FusionArt_YYYYMMDD_HHmmss.png` (ej. `FusionArt_20260422_143022.png`) |
| `MIME_TYPE`                    | String    | `image/png` (o `image/jpeg` según el formato elegido)         |
| `RELATIVE_PATH`                | String    | `Pictures/FusionArtIA/` (para Android 10+)                    |
| `DATE_ADDED`                   | Long      | Automático (segundos desde epoch)                              |
| `DATE_MODIFIED`                | Long      | Automático                                                     |
| `SIZE`                         | Long      | Automático (tamaño del archivo en bytes)                      |
| `IS_PENDING`                   | Int (0/1) | Se usa `1` mientras se escribe el archivo, luego `0`           |
| `OWNER_PACKAGE_NAME`           | String    | Nombre del paquete de la app (automático)                      |

### 2.3. Código de inserción (Kotlin)

```kotlin
val contentValues = ContentValues().apply {
    put(MediaStore.Images.Media.DISPLAY_NAME, fileName)
    put(MediaStore.Images.Media.MIME_TYPE, "image/png")
    put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/FusionArtIA/")
}

val uri = contentResolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues)
uri?.let {
    contentResolver.openOutputStream(it)?.use { outputStream ->
        fusedBitmap.compress(Bitmap.CompressFormat.PNG, 100, outputStream)
    }
    // Finalizar escritura
    contentValues.clear()
    put(MediaStore.Images.Media.IS_PENDING, 0)
    contentResolver.update(it, contentValues, null, null)
}
```

---

## 3. Preferencias de la aplicación (DataStore)

**No implementado en el prototipo**, pero se define el esquema para futuras versiones. Se usaría `Preferences DataStore` (clave-valor).

### 3.1. Claves definidas

| Clave                 | Tipo      | Valor por defecto | Descripción                                      |
| --------------------- | --------- | ----------------- | ------------------------------------------------ |
| `api_provider`        | String    | `"gemini"`        | Proveedor de IA: `"gemini"` o `"firefly"`.       |
| `image_quality`       | Int       | `85`              | Calidad de compresión JPEG (0-100).              |
| `auto_save`           | Boolean   | `false`           | Guardar automáticamente tras fusión exitosa.     |
| `last_used_prompt`    | String    | (prompt por defecto) | Texto del último prompt usado.                 |

### 3.2. Ejemplo de acceso (para referencia)

```kotlin
val Context.dataStore by preferencesDataStore(name = "fusionart_settings")

val apiProviderFlow: Flow<String> = context.dataStore.data
    .map { preferences ->
        preferences[PreferencesKeys.API_PROVIDER] ?: "gemini"
    }

suspend fun saveApiProvider(provider: String) {
    context.dataStore.edit { settings ->
        settings[PreferencesKeys.API_PROVIDER] = provider
    }
}
```

---

## 4. Almacenamiento temporal (en memoria)

El prototipo **no persiste** las imágenes seleccionadas entre sesiones. Los datos que viven en RAM son:

| Dato                          | Tipo               | Ciclo de vida                          |
| ----------------------------- | ------------------ | -------------------------------------- |
| URI de fondo                  | `Uri`              | Hasta que se limpia o se cierra la app |
| URI de primer plano           | `Uri`              | Hasta que se limpia o se cierra la app |
| Miniatura de fondo (opcional) | `Bitmap` (pequeño) | Mismo ciclo (se libera con Coil)       |
| Miniatura de primer plano     | `Bitmap` (pequeño) | Mismo ciclo                            |
| Imagen fusionada (resultado)  | `Bitmap` (grande)  | Mientras está en ResultScreen          |

Al rotar la pantalla, las URIs se guardan en `SavedStateHandle` del ViewModel.

---

## 5. Diagrama de persistencia (textual)

```
┌─────────────────────────────────────────────────────────────┐
│                      Dispositivo Android                     │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                     Memoria RAM                        │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  │  │
│  │  │ URIs fondo  │  │ URIs primer │  │ Bitmap fusion │  │  │
│  │  │ y primer    │  │ plano       │  │ (temporal)    │  │  │
│  │  └─────────────┘  └─────────────┘  └───────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Almacenamiento persistente               │  │
│  │  ┌─────────────────┐      ┌────────────────────────┐ │  │
│  │  │   MediaStore    │      │  DataStore (prefs)      │ │  │
│  │  │ (imagen final)  │      │  (api_provider, etc.)   │ │  │
│  │  └─────────────────┘      └────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. Permisos relacionados con el esquema

| Permiso                              | Necesario para               | Versión de Android                     |
| ------------------------------------ | ---------------------------- | -------------------------------------- |
| `READ_MEDIA_IMAGES`                  | Leer imágenes de la galería  | Android 13+ (API 33)                   |
| `READ_EXTERNAL_STORAGE`              | Leer imágenes                | Android 12 y anteriores (API 24-32)    |
| `WRITE_EXTERNAL_STORAGE`             | Guardar imagen (si no se usa MediaStore con RELATIVE_PATH) | Android 9 y anteriores (API 24-28) |
| Ninguno (MediaStore)                 | Guardar en Android 10+       | API 29+ (no se necesita permiso de escritura para su propia carpeta) |

En el prototipo se usará `MediaStore` con `RELATIVE_PATH`, por lo que **no se requiere permiso de escritura en Android 10+**.

---

## 7. Resumen de esquemas

| Tipo de almacenamiento | Tecnología      | ¿Usado en prototipo? | Contenido                               |
| ---------------------- | --------------- | -------------------- | --------------------------------------- |
| Base de datos          | Room            | ❌ No                | -                                       |
| Archivos de imagen     | MediaStore      | ✅ Sí                | Imagen fusionada guardada por el usuario |
| Preferencias           | DataStore       | ❌ No (pero definido) | Configuración futura (proveedor IA, calidad) |
| Memoria temporal       | Variables Kotlin| ✅ Sí                | URIs, bitmaps, estado de la UI          |

---

> **Nota:** Este esquema es mínimo y suficiente para el prototipo. Si se evoluciona a una versión completa, se podría añadir Room para guardar historial de fusiones (fecha, imágenes originales, resultado, prompt usado).