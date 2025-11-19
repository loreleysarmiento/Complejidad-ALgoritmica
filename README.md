# SkyRoute - Optimización de Rutas Aéreas

Aplicación Android para calcular y optimizar rutas aéreas en Sudamérica, desarrollada con Jetpack Compose y Firebase.

## 🚀 Características

- ✅ **Autenticación**: Login y registro de usuarios con Firebase Auth
- ✅ **Búsqueda de Rutas**: Calcula rutas entre aeropuertos con opciones:
  - Ruta más rápida
  - Ruta más económica
- ✅ **Historial**: Guarda y visualiza todas las rutas calculadas
- ✅ **Perfil de Usuario**: Gestión de información personal y foto de perfil
- ✅ **Diseño Moderno**: Tema personalizado con colores de aviación y Material Design 3

## 🏗️ Arquitectura

- **UI**: Jetpack Compose con Material 3
- **Navegación**: Navigation Compose
- **Estado**: ViewModels con StateFlow
- **Backend**: Firebase (Auth, Firestore, Storage)
- **Preparado para API REST**: Retrofit configurado para conectar con backend personalizado

## 📋 Requisitos Previos

- Android Studio Hedgehog (2023.1.1) o superior
- JDK 11 o superior
- SDK mínimo: Android 7.0 (API 24)
- SDK objetivo: Android 14 (API 34)

## 🔧 Configuración de Firebase

### 1. Crear proyecto en Firebase

1. Ve a [Firebase Console](https://console.firebase.google.com/)
2. Crea un nuevo proyecto llamado "SkyRoute" o el nombre que prefieras
3. Habilita Google Analytics (opcional)

### 2. Registrar la aplicación Android

1. En la consola de Firebase, haz clic en "Agregar aplicación" → Android
2. Ingresa el nombre del paquete: `com.example.complejidad`
3. Descarga el archivo `google-services.json`
4. **Reemplaza** el archivo `app/google-services.json` existente con el que descargaste

### 3. Habilitar servicios de Firebase

#### Authentication
1. Ve a **Authentication** → **Sign-in method**
2. Habilita **Email/Password**

#### Firestore Database
1. Ve a **Firestore Database** → **Crear base de datos**
2. Selecciona modo **Producción**
3. Elige la ubicación más cercana a tu región
4. Actualiza las reglas de seguridad:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Usuarios solo pueden leer/escribir sus propios datos
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Rutas - usuarios solo pueden crear y leer sus propias rutas
    match /routes/{routeId} {
      allow create: if request.auth != null;
      allow read: if request.auth != null && resource.data.userId == request.auth.uid;
    }
  }
}
```

#### Storage
1. Ve a **Storage** → **Comenzar**
2. Selecciona modo **Producción**
3. Actualiza las reglas de seguridad:

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /profile_photos/{userId}.jpg {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## 📱 Instalación y Ejecución

1. **Clonar el repositorio** (si aplica) o abrir el proyecto en Android Studio

2. **Configurar Firebase** siguiendo los pasos anteriores

3. **Sincronizar Gradle**:
   ```
   Build → Make Project
   ```

4. **Ejecutar la aplicación**:
   - Conecta un dispositivo Android o inicia un emulador
   - Haz clic en Run (▶️)

## 🎨 Estructura del Proyecto

```
app/src/main/java/com/example/complejidad/
├── data/
│   ├── model/          # Modelos de datos (User, Airport, Route)
│   └── repository/     # Repositorios (Auth, User, Route)
├── navigation/         # Navegación de la app
├── ui/
│   ├── screens/
│   │   ├── auth/      # Pantallas de Login y Registro
│   │   └── main/      # Home, Historial, Perfil
│   └── theme/         # Tema y colores personalizados
├── viewmodel/         # ViewModels para gestión de estado
└── MainActivity.kt
```

## 🔌 Integración con Backend Personalizado

La aplicación está preparada para conectarse con un backend REST. Para integrar tu backend:

### 1. Configurar la URL base

En `RouteRepository.kt`, actualiza para usar Retrofit:

```kotlin
private val retrofit = Retrofit.Builder()
    .baseUrl("https://tu-backend-url.com/api/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

private val apiService = retrofit.create(RouteApiService::class.java)
```

### 2. Crear interfaz de API

```kotlin
interface RouteApiService {
    @POST("routes/calculate")
    suspend fun calculateRoute(
        @Body request: RouteRequest
    ): Response<RouteResponse>
    
    @GET("airports")
    suspend fun getAirports(): Response<List<Airport>>
}
```

### 3. Endpoints esperados del backend

- **POST /routes/calculate**
  - Body: `{ "origin": "LIM", "destination": "EZE", "routeType": "FASTEST" }`
  - Response: `{ "distance": 3120.5, "time": 3.9, "cost": 374.4, "stops": [...] }`

- **GET /airports**
  - Response: `[{ "id": "1", "name": "...", "iataCode": "LIM", ... }]`

## 📊 Datos de Ejemplo

La app incluye 6 aeropuertos principales de Sudamérica:
- Lima (LIM) - Perú
- Bogotá (BOG) - Colombia
- Buenos Aires (EZE) - Argentina
- São Paulo (GRU) - Brasil
- Santiago (SCL) - Chile
- Quito (UIO) - Ecuador

## 🎯 Funcionalidades Principales

### 1. Autenticación
- Registro con email, nombre y contraseña
- Login con validación
- Cierre de sesión seguro

### 2. Búsqueda de Rutas
- Selección de aeropuerto de origen y destino
- Dos tipos de optimización:
  - **Más Rápida**: Minimiza tiempo de viaje
  - **Más Económica**: Minimiza costo (puede incluir escalas)
- Cálculo usando fórmula de Haversine para distancias

### 3. Historial
- Visualización de todas las rutas calculadas
- Ordenadas por fecha (más recientes primero)
- Detalles completos: distancia, tiempo, costo

### 4. Perfil
- Edición de nombre y email
- Cambio de foto de perfil (desde galería)
- Foto predeterminada generada automáticamente
- Opción de cerrar sesión

## 🎨 Tema de Diseño

- **Color principal**: Azul Aviación (`#0A4D8C`)
- **Acento**: Naranja (`#FF9800`) y Verde (`#4CAF50`)
- **Tema claro/oscuro** soportado
- **Material Design 3** con componentes modernos

## 🔄 Próximos Pasos

- [ ] Conectar con backend real para cálculo de rutas con Dijkstra/Bellman-Ford
- [ ] Agregar visualización de mapa con rutas
- [ ] Implementar filtros avanzados (escalas, aerolíneas, etc.)
- [ ] Agregar notificaciones push
- [ ] Soporte offline con Room Database
- [ ] Análisis de rutas favoritas y estadísticas

## 🐛 Solución de Problemas

### Error: "Default FirebaseApp is not initialized"
- Verifica que `google-services.json` esté en la carpeta `app/`
- Asegúrate de que el plugin de Google Services esté aplicado en `build.gradle.kts`

### Error de compilación con Compose
- Limpia el proyecto: `Build → Clean Project`
- Invalida caché: `File → Invalidate Caches / Restart`

### Imágenes no cargan
- Verifica permisos en AndroidManifest.xml
- Comprueba las reglas de Storage en Firebase

## 📄 Licencia

Proyecto académico para análisis de rutas aéreas en Sudamérica.

## 👥 Contacto

Para dudas o sugerencias sobre el proyecto, contacta al equipo de desarrollo.

---

**Nota**: Este es un proyecto en desarrollo. El archivo `google-services.json` incluido es un placeholder y debe ser reemplazado con tu configuración real de Firebase.
