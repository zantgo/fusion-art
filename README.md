# FusionArt IA

**FusionArt IA** es una aplicación Android (prototipo) que permite fusionar una imagen de fondo y una imagen de primer plano utilizando inteligencia artificial. La aplicación envía ambas imágenes a una API de IA (Google Gemini 2.5 Flash o Adobe Firefly) y devuelve una nueva imagen donde el objeto del primer plano se integra de forma coherente (iluminación, sombras, perspectiva) sobre el fondo seleccionado.

> 📱 **Estado del proyecto:** Prototipo funcional. Desarrollado en Kotlin + Jetpack Compose.

---

## 📋 Tabla de contenidos

- [Características principales](#-características-principales)
- [Arquitectura y documentación técnica](#-arquitectura-y-documentación-técnica)
- [Requisitos previos](#-requisitos-previos)
- [Instalación y configuración](#-instalación-y-configuración)
- [Cómo usar la app](#-cómo-usar-la-app)
- [Estructura del proyecto](#-estructura-del-proyecto)
- [Licencia](#-licencia)
- [Contacto](#-contacto)

---

## ✨ Características principales

- Selección de imagen de **fondo** y de **primer plano** desde la galería.
- Previsualización de ambas imágenes antes de fusionar.
- Llamada a API de IA (soporte inicial para **Google Gemini 2.5 Flash**).
- Indicador de progreso durante la fusión.
- Visualización de la imagen resultante.
- Guardado de la imagen fusionada en la galería del dispositivo.
- Manejo básico de errores (red, formato, tiempo de espera).

---

## 📚 Arquitectura y documentación técnica

El proyecto sigue una arquitectura limpia (Clean Architecture) simplificada. Toda la especificación del sistema se encuentra desglosada en los siguientes documentos:

| Documento | Descripción |
|-----------|-------------|
| [**Arquitectura del software**](./docs/architecture.md) | Capas, patrones, decisiones técnicas y diagrama de componentes. |
| [**Diseño del sistema**](./docs/system-design.md) | Diagramas de secuencia, despliegue y explicación de los módulos principales. |
| [**Diseño de UI/UX**](./docs/ux-design.md) | Paleta de colores, tipografía, componentes visuales y guía de estilos. |
| [**Flujo de usuario (UX Flow)**](./docs/ux-flow.md) | Diagrama de navegación entre pantallas y estados de la interfaz. |
| [**Casos de uso**](./docs/use-cases.md) | Descripción detallada de cada interacción usuario-sistema (formato CUxxx). |
| [**Referencia de la API**](./docs/api-reference.md) | Endpoints, formatos de petición/respuesta, códigos de error y ejemplos. |
| [**Modelo de datos**](./docs/data-model.md) | Entidades (ImageData, FusionState), relaciones y contratos. |
| [**Esquema de la base de datos local**](./docs/schema.md) | Tablas (si las hubiera) y preferencias con DataStore. |
| [**Casos de prueba**](./docs/test-cases.md) | Pruebas unitarias, de integración y de UI, con escenarios esperados. |

> 💡 **Nota:** Todos los documentos están escritos en español y siguen el estándar de especificación definido en el [Documento de Especificación del Sistema](./docs/system-specification.md) (entregado previamente).

---

## ⚙️ Requisitos previos

- **Android Studio** Jellyfish | 2023.3.1 o superior.
- **JDK 17**.
- **SDK de Android** mínimo API 24 (Android 7.0).
- **Una clave de API** de Google Gemini (o Adobe Firefly).
- **Dispositivo o emulador** con Android 7.0+ y acceso a internet.

---

## 🛠 Instalación y configuración

1. **Clonar el repositorio**
   ```bash
   git clone https://github.com/tu-usuario/fusionart-ia.git
   cd fusionart-ia
   ```

2. **Abrir el proyecto** en Android Studio.

3. **Configurar la API Key**
   - Crea un archivo `local.properties` en la raíz del proyecto (si no existe).
   - Añade la siguiente línea:
     ```properties
     GEMINI_API_KEY=tu_clave_aqui
     ```
   - En código, la clave se accede mediante `BuildConfig.GEMINI_API_KEY`.

4. **Sincronizar el proyecto** (Gradle Sync).

5. **Ejecutar la app** en un dispositivo o emulador.

---

## 📱 Cómo usar la app

1. Abre la aplicación **FusionArt IA**.
2. Toca el botón **"Seleccionar fondo"** y elige una imagen de tu galería.
3. Toca el botón **"Seleccionar primer plano"** y elige otra imagen.
4. Una vez cargadas ambas, pulsa el botón **"Fusionar"**.
5. Espera unos segundos mientras la IA procesa las imágenes.
6. En la pantalla de resultado, puedes:
   - **Guardar** la imagen fusionada en tu galería.
   - **Volver** a empezar una nueva fusión.
7. Si ocurre un error (sin internet, formato no válido, etc.), se mostrará un mensaje explicativo.

---

## 🗂 Estructura del proyecto

```
fusionart-ia/
├── app/
│   ├── src/main/java/com/fusionart/ia/
│   │   ├── data/               # Capa de datos (API, repositorios)
│   │   ├── domain/             # Casos de uso y modelos
│   │   ├── presentation/       # UI (Compose), ViewModels
│   │   └── utils/              # Utilidades (permisos, redimensionado)
│   ├── res/                    # Recursos (drawable, values, etc.)
│   └── build.gradle.kts
├── docs/                       # Documentación técnica
│   ├── architecture.md
│   ├── system-design.md
│   ├── ux-design.md
│   ├── ux-flow.md
│   ├── use-cases.md
│   ├── api-reference.md
│   ├── data-model.md
│   ├── schema.md
│   └── test-cases.md
├── local.properties            # (ignorado por git) Claves de API
├── README.md                   # Este archivo
└── build.gradle.kts (proyecto)
