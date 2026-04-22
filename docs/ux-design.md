# Diseño de UI/UX – FusionArt IA (Prototipo)

## 1. Objetivos de experiencia de usuario

- **Simplicidad**: El usuario debe poder fusionar dos imágenes en menos de 3 pasos.
- **Claridad**: Las imágenes seleccionadas se previsualizan claramente antes de fusionar.
- **Feedback inmediato**: Indicadores visuales durante la carga y mensajes de error comprensibles.
- **Resultado visible**: La imagen fusionada se muestra a tamaño completo con opción de guardar.

---

## 2. Perfiles de usuario (proto-personas)

| Persona | Edad | Contexto | Necesidad principal |
|---------|------|----------|---------------------|
| **Laura** (diseñadora aficionada) | 28 | Crea contenido para redes sociales | Quiere combinar objetos de una foto con fondos atractivos sin usar Photoshop. |
| **Carlos** (desarrollador) | 35 | Prueba herramientas de IA | Evalúa la calidad de fusión de la API. Valora la velocidad y la integración técnica. |
| **Marta** (usuario casual) | 45 | Quiere hacer montajes familiares | Necesita una interfaz intuitiva, sin configuraciones complejas. |

---

## 3. Flujo de pantallas (estructura)

La aplicación tiene **dos pantallas principales**:

1. **Pantalla de selección (MainScreen)**
2. **Pantalla de resultado (ResultScreen)**

No hay navegación compleja (sin drawer, sin tabs). El botón "Nueva fusión" regresa a la pantalla de selección.

---

## 4. Wireframes (descripción textual)

### 4.1 Pantalla de selección (MainScreen)

```
-------------------------------------------------
|                  FusionArt IA                |
-------------------------------------------------
|                                               |
|   ┌─────────────────────────────────────┐     |
|   |                                     |     |
|   |       IMAGEN DE FONDO               |     |
|   |     (vista previa o placeholder)    |     |
|   |                                     |     |
|   └─────────────────────────────────────┘     |
|          [ Seleccionar fondo ]                |
|                                               |
|   ┌─────────────────────────────────────┐     |
|   |                                     |     |
|   |     IMAGEN DE PRIMER PLANO          |     |
|   |     (vista previa o placeholder)    |     |
|   |                                     |     |
|   └─────────────────────────────────────┘     |
|          [ Seleccionar primer plano ]        |
|                                               |
|           [      FUSIONAR      ]              |
|           [   Limpiar todo    ]              |
|                                               |
-------------------------------------------------
```

**Elementos**:
- Dos tarjetas rectangulares (aspect ratio 16:9) con fondo gris claro y texto centrado.
- Botones redondeados debajo de cada tarjeta.
- Botón principal "Fusionar" en color primario, deshabilitado hasta tener ambas imágenes.
- Botón secundario "Limpiar todo" (texto, sin relleno).

### 4.2 Pantalla de carga (modal)

Aparece como un diálogo o pantalla completa con:

```
┌─────────────────────────┐
│      ⏳ (indicador)      │
│   Fusionando con IA...   │
│   (texto opcional)       │
└─────────────────────────┘
```

- Fondo semitransparente.
- Impide interacción con la pantalla subyacente.
- No tiene botón de cancelar en el prototipo.

### 4.3 Pantalla de resultado (ResultScreen)

```
-------------------------------------------------
|                  FusionArt IA                 |
-------------------------------------------------
|                                               |
|   ┌─────────────────────────────────────┐     |
|   │                                     │     |
|   │                                     │     |
|   │        IMAGEN FUSIONADA             │     |
|   │        (tamaño completo)            │     |
|   │                                     │     |
|   └─────────────────────────────────────┘     |
|                                               |
|        [    GUARDAR EN GALERÍA    ]           |
|        [      NUEVA FUSIÓN        ]           |
|                                               |
-------------------------------------------------
```

- La imagen ocupa la mayor parte de la pantalla (ancho completo, alto ajustable).
- Botones grandes y centrados.
- Opcional: mensaje de éxito tras guardar (Snackbar).

---

## 5. Paleta de colores (Material 3)

| Rol                 | Color (hex)  | Uso                              |
|---------------------|--------------|----------------------------------|
| Primary             | #6750A4      | Botón "Fusionar", elementos activos |
| On Primary          | #FFFFFF      | Texto sobre primary              |
| Secondary           | #625B71      | Botón "Nueva fusión"             |
| Surface             | #FFFBFE      | Fondo general                    |
| Surface Variant     | #E7E0EC      | Tarjetas de imágenes             |
| Error               | #BA1A1A      | Mensajes de error                |
| Outline             | #79747E      | Bordes de tarjetas               |
| Text Primary        | #1C1B1F      | Títulos y texto principal        |
| Text Secondary      | #49454F      | Texto de ayuda                   |

**Modo oscuro**: Se invertirán los roles de surface y on surface, manteniendo primary.

---

## 6. Tipografía

| Estilo             | Fuente     | Tamaño (sp) | Peso     | Uso                          |
|--------------------|------------|-------------|----------|------------------------------|
| HeadlineLarge      | Roboto     | 24          | Medium   | Título de la app             |
| TitleMedium        | Roboto     | 18          | Medium   | Encabezados de tarjeta       |
| BodyLarge          | Roboto     | 16          | Regular  | Texto de botones             |
| BodyMedium         | Roboto     | 14          | Regular  | Texto de ayuda, mensajes     |
| LabelLarge         | Roboto     | 14          | Medium   | Etiquetas de campo           |

---

## 7. Componentes de UI (guía de diseño)

### 7.1 Botón primario
- Fondo: Primary (#6750A4)
- Texto: Blanco, BodyLarge
- Esquinas redondeadas (12dp)
- Altura: 52dp
- Estado deshabilitado: fondo gris (#BDBDBD), texto gris oscuro

### 7.2 Botón secundario
- Sin fondo, solo texto Primary
- Borde opcional (OutlinedButton de Material)

### 7.3 Tarjeta de imagen
- Fondo: Surface Variant (#E7E0EC)
- Borde redondeado (16dp)
- Relación de aspecto 16:9
- Contenido centrado: ícono de imagen (o miniatura cargada)
- Sombra suave (elevación 2dp)

### 7.4 Indicador de progreso
- CircularProgressIndicator (color Primary)
- Tamaño 48dp
- Acompañado de texto "Fusionando..."

### 7.5 Snackbar (mensajes temporales)
- Fondo oscuro (#333333)
- Texto blanco
- Duración: 3 segundos
- Usado para: "Imagen guardada en galería", "Error de red"

---

## 8. Microinteracciones

| Acción                         | Respuesta visual                                                                 |
|--------------------------------|----------------------------------------------------------------------------------|
| Tocar botón "Seleccionar fondo"| Se abre la galería del sistema (transición estándar)                            |
| Imagen cargada en tarjeta      | Animación de fade in (300ms) de la miniatura                                    |
| Tocar botón "Fusionar" (habilitado) | Botón se escala ligeramente (0.95), luego muestra indicador de carga        |
| Fusión completada              | Transición deslizante de derecha a izquierda hacia ResultScreen                 |
| Tocar "Guardar"                | Animación de éxito (checkmark) breve + Snackbar "Guardado"                      |
| Error de red                   | Snackbar rojo con mensaje, el botón "Fusionar" se rehabilita                    |

---

## 9. Accesibilidad (consideraciones básicas)

- **Contraste**: Cumple WCAG 2.1 AA (relación de contraste mínima 4.5:1 para texto).
- **Tamaño de objetivo táctil**: Todos los botones tienen al menos 48x48dp.
- **Texto alternativo**: Las imágenes no tienen contenido textual, pero los botones usan `contentDescription`.
- **Escalado de texto**: Soporte para fuentes del sistema hasta 1.5x.

---

## 10. Estados de la interfaz

| Estado                     | Elementos visibles                                 |
|----------------------------|----------------------------------------------------|
| **Inicial**                | Placeholders, botones de selección, botón Fusionar deshabilitado |
| **Fondo seleccionado**     | Miniatura de fondo, botón "Seleccionar fondo" cambia a "Cambiar fondo" |
| **Ambas seleccionadas**    | Ambas miniaturas, botón Fusionar habilitado        |
| **Cargando**               | Overlay con progreso, botones deshabilitados       |
| **Resultado**              | Imagen fusionada, botones Guardar y Nueva fusión   |
| **Error**                  | Snackbar de error, botón Fusionar rehabilitado     |

---

## 11. Maquetación responsive

- La app se adapta a **teléfonos** (retrato principalmente).
- En **tabletas**, las tarjetas se muestran en una columna centrada con ancho máximo de 600dp.
- En **landscape**, las dos tarjetas pueden mostrarse en fila si el ancho lo permite (opcional para prototipo, no crítico).

---

## 12. Iconografía

| Icono                | Fuente               | Uso                          |
|----------------------|----------------------|------------------------------|
| `photo_library`      | Material Icons       | Botón seleccionar fondo      |
| `image`              | Material Icons       | Placeholder de imagen        |
| `merge_type`         | Material Icons       | Botón Fusionar               |
| `save`               | Material Icons       | Botón Guardar                |
| `refresh`            | Material Icons       | Botón Nueva fusión           |
| `delete`             | Material Icons       | Botón Limpiar todo           |

---

## 13. Resumen de pantallas y transiciones

```
[MainScreen] 
    │ (seleccionar imágenes)
    │ (pulsar Fusionar)
    ▼
[Loading overlay] 
    │ (éxito)
    ▼
[ResultScreen] 
    │ (Guardar → Snackbar)
    │ (Nueva fusión → MainScreen con estado limpio)
    ▼
[MainScreen]
```

---

## 14. Notas para el desarrollador

- Usar `MaterialTheme` de Compose con colores personalizados.
- Las miniaturas se cargan con Coil (`.subcompose` para placeholder).
- El estado de carga se maneja con un `Dialog` o un `Box` con fondo semitransparente.
- Las transiciones entre pantallas se implementan con `NavHost` de Compose Navigation.
