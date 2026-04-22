# Flujo de usuario (UX Flow) – FusionArt IA (Prototipo)

## 1. Objetivo del documento

Describir la secuencia de pantallas y las acciones que realiza el usuario para completar la tarea principal: **fusionar dos imágenes usando IA**. También se cubren los caminos alternativos (errores, cancelaciones, reinicios).

---

## 2. Diagrama de flujo principal (textual)

```
┌─────────────────┐
│   Inicio app    │
└────────┬────────┘
         ▼
┌─────────────────────────────────────┐
│  Pantalla de selección (MainScreen) │
│  - Placeholders vacíos              │
│  - Botón "Fusionar" deshabilitado   │
└────────┬────────────────────────────┘
         │
         ▼ (usuario toca "Seleccionar fondo")
┌─────────────────────────────────────┐
│  Selector de galería (sistema)      │
│  - Usuario elige una imagen         │
└────────┬────────────────────────────┘
         │
         ▼ (imagen devuelta)
┌─────────────────────────────────────┐
│  MainScreen actualizada             │
│  - Miniatura de fondo visible       │
│  - Botón "Cambiar fondo" opcional   │
│  - Si ya hay primer plano → botón   │
│    "Fusionar" se habilita           │
└────────┬────────────────────────────┘
         │
         ▼ (usuario toca "Seleccionar primer plano")
┌─────────────────────────────────────┐
│  Selector de galería (sistema)      │
│  - Usuario elige otra imagen        │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  MainScreen con ambas miniaturas    │
│  - Botón "Fusionar" habilitado      │
└────────┬────────────────────────────┘
         │
         ▼ (usuario toca "Fusionar")
┌─────────────────────────────────────┐
│  Estado de carga (overlay)          │
│  - Indicador circular               │
│  - Texto "Fusionando con IA..."     │
└────────┬────────────────────────────┘
         │
         ├─── (éxito) ───► ┌─────────────────────────┐
         │                  │ Pantalla de resultado   │
         │                  │ - Imagen fusionada      │
         │                  │ - Botones Guardar /     │
         │                  │   Nueva fusión          │
         │                  └────────────┬────────────┘
         │                               │
         │                               ├── (Guardar) ──► Snackbar "Guardado"
         │                               │
         │                               └── (Nueva fusión) ──► MainScreen limpia
         │
         └─── (error) ───► ┌─────────────────────────┐
                            │  MainScreen con error   │
                            │  - Snackbar rojo con    │
                            │    mensaje explicativo  │
                            │  - Botón Fusionar       │
                            │    rehabilitado         │
                            └─────────────────────────┘
```

---

## 3. Descripción detallada de cada paso

### Paso 0: Inicio de la aplicación
- **Pantalla**: MainScreen.
- **Estado inicial**: Ambas tarjetas muestran un ícono de imagen y el texto "Sin imagen".
- **Botones**: "Seleccionar fondo", "Seleccionar primer plano", "Limpiar todo" (visible pero inactivo hasta tener alguna imagen). "Fusionar" deshabilitado (gris).

### Paso 1: Seleccionar imagen de fondo
- **Acción**: Usuario toca "Seleccionar fondo".
- **Comportamiento**: Se abre el selector nativo de archivos (imágenes). Filtro por `image/*`.
- **Si elige una imagen**: Se muestra miniatura en la tarjeta de fondo. El texto del botón cambia a "Cambiar fondo".
- **Si cancela**: No hay cambio, el estado permanece igual.

### Paso 2: Seleccionar imagen de primer plano
- **Acción**: Usuario toca "Seleccionar primer plano".
- **Comportamiento**: Mismo selector de galería.
- **Tras elegir**: Se muestra miniatura en la tarjeta de primer plano. El botón "Fusionar" se **habilita** (cambia a color primario).

### Paso 3: Fusión (camino feliz)
- **Acción**: Usuario toca "Fusionar".
- **Comportamiento**:
  - Botón "Fusionar" se deshabilita temporalmente.
  - Aparece un overlay de carga (diálogo o pantalla modal) que bloquea la interacción.
  - La app redimensiona las imágenes, las convierte a Base64 y llama a la API de Gemini.
- **Mientras tanto**: El usuario no puede hacer nada más; si pulsa atrás, el diálogo debería cerrarse (opcional, pero en prototipo se permite cancelar la operación? Decisión: **no** para simplificar).
- **Al recibir respuesta exitosa**:
  - Se cierra el overlay.
  - Se navega a ResultScreen pasando el Bitmap de la imagen fusionada.

### Paso 4: Visualizar resultado
- **Pantalla**: ResultScreen.
- **Contenido**: Imagen fusionada ocupando la mayor parte de la pantalla.
- **Botones**:
  - "Guardar en galería" (primario)
  - "Nueva fusión" (secundario)

### Paso 5: Guardar la imagen
- **Acción**: Usuario toca "Guardar en galería".
- **Comportamiento**:
  - Se solicita permiso de escritura si es necesario (Android <10).
  - Se guarda la imagen en la carpeta `Pictures/FusionArtIA/`.
  - Se muestra un Snackbar: "Imagen guardada en galería".
  - (Opcional) Se reproduce una breve animación de éxito.

### Paso 6: Nueva fusión
- **Acción**: Usuario toca "Nueva fusión".
- **Comportamiento**:
  - Se limpia el estado del ViewModel (borrar URIs y bitmaps).
  - Se navega de vuelta a MainScreen con todas las tarjetas vacías.

---

## 4. Caminos alternativos (casos de error)

### Error 1: Sin conexión a internet
- **Disparador**: El usuario pulsa "Fusionar" estando offline.
- **Comportamiento**:
  - Se muestra Snackbar en MainScreen: "No hay conexión a internet. Conéctate e inténtalo de nuevo".
  - El botón "Fusionar" se rehabilita.
  - No se muestra overlay de carga.

### Error 2: Tiempo de espera agotado (timeout >15s)
- **Disparador**: La API no responde en 15 segundos.
- **Comportamiento**:
  - Se cierra el overlay de carga.
  - Snackbar: "La IA tardó demasiado. Inténtalo de nuevo más tarde".
  - El botón "Fusionar" se rehabilita en MainScreen.

### Error 3: API devuelve error (4xx o 5xx)
- **Disparador**: Clave inválida, formato no soportado, etc.
- **Comportamiento**:
  - Overlay cerrado.
  - Snackbar con mensaje específico (ej. "Error de autenticación. Contacta soporte").
  - Se mantienen las imágenes seleccionadas; usuario puede reintentar.

### Error 4: Formato de imagen no válido
- **Disparador**: El usuario selecciona un archivo que no es imagen (aunque el selector filtra, podría ser un archivo corrupto).
- **Comportamiento**:
  - Al intentar cargar la miniatura con Coil, se captura la excepción.
  - Snackbar: "No se pudo cargar la imagen. Elige otra".

### Error 5: Memoria insuficiente al decodificar resultado
- **Disparador**: La imagen fusionada es demasiado grande (raro porque Gemini suele devolver 1024x1024).
- **Comportamiento**:
  - Se captura `OutOfMemoryError`.
  - Snackbar: "La imagen resultante es demasiado pesada. Prueba con imágenes más pequeñas".

---

## 5. Flujo de "Limpiar todo"

- **Disponible**: Siempre que al menos una imagen esté seleccionada.
- **Acción**: Tocar "Limpiar todo".
- **Comportamiento**:
  - Borra ambas imágenes seleccionadas.
  - Las tarjetas vuelven a placeholder.
  - Botón "Fusionar" se deshabilita.
  - Botón "Limpiar todo" se deshabilita (o se oculta).

---

## 6. Transiciones entre pantallas

| Origen          | Destino          | Animación sugerida                       |
|-----------------|------------------|------------------------------------------|
| MainScreen      | ResultScreen     | Deslizamiento hacia la izquierda (slide_in_left) |
| ResultScreen    | MainScreen       | Deslizamiento hacia la derecha (pop)     |
| MainScreen      | (overlay)        | Fade in                                  |
| (overlay)       | MainScreen/Error | Fade out                                 |

---

## 7. Estados del botón "Fusionar"

| Condición                                | Estado        | Color        |
|------------------------------------------|---------------|--------------|
| Fondo vacío O primer plano vacío         | Deshabilitado | Gris (#BDBDBD) |
| Ambas imágenes seleccionadas             | Habilitado    | Primary (#6750A4) |
| Durante la llamada a la API              | Deshabilitado (y oculto tras overlay) | - |

---

## 8. Manejo de rotación y cambios de configuración

- **URIs seleccionadas**: Se guardan en `SavedStateHandle` del ViewModel para que no se pierdan al rotar.
- **Bitmap de resultado**: Se guarda de la misma forma (o se almacena en caché en memoria).
- **Estado de carga**: Se mantiene; si la rotación ocurre mientras carga, el overlay se vuelve a mostrar.

---

## 9. Resumen de puntos de decisión del usuario

| Pantalla                | Decisión del usuario                          |
|-------------------------|-----------------------------------------------|
| MainScreen              | Elegir fondo y primer plano                   |
| MainScreen              | Pulsar Fusionar o Limpiar todo                |
| Selector de galería     | Elegir imagen o cancelar                      |
| ResultScreen            | Guardar o Nueva fusión                        |
| (cualquiera)            | Botón Atrás del sistema (cierra la app o vuelve según contexto) |
