# 📋 Notas Técnicas del Proyecto

## 🏗️ Arquitectura

### Patrón MVVM (Model-View-ViewModel)

```
UI Layer (Compose)
    ↓
ViewModel (StateFlow)
    ↓
Repository
    ↓
Data Sources (Firebase / API)
```

### Estructura de Paquetes

```
com.example.complejidad/
├── data/
│   ├── model/              # Data classes (User, Airport, Route)
│   └── repository/         # Lógica de acceso a datos
├── navigation/             # Configuración de navegación
├── ui/
│   ├── screens/
│   │   ├── auth/          # Login, Register
│   │   └── main/          # Home, History, Profile
│   └── theme/             # Colores, tipografía
├── viewmodel/             # Lógica de presentación
└── MainActivity.kt        # Entry point
```

## 🔄 Flujo de Datos

### 1. Autenticación
```
LoginScreen → AuthViewModel → AuthRepository → Firebase Auth → Success → Navigate to Home
```

### 2. Cálculo de Ruta
```
HomeScreen → RouteViewModel → RouteRepository → [Simulación/API] → Firestore → StateFlow → UI
```

### 3. Historial
```
HistoryScreen → RouteViewModel → RouteRepository → Firestore Query → List<Route> → UI
```

### 4. Perfil
```
ProfileScreen → UserViewModel → UserRepository → Firestore/Storage → User → UI
```

## 🎨 Sistema de Diseño

### Colores Principales
- **Primary**: `#0A4D8C` (Azul Aviación)
- **Secondary**: `#4FC3F7` (Azul Cielo)
- **Tertiary**: `#FF9800` (Naranja Acento)
- **Success**: `#4CAF50` (Verde)
- **Error**: `#F44336` (Rojo)

### Componentes Reutilizables
- `RouteResultCard`: Muestra resultado de ruta calculada
- `HistoryRouteCard`: Item de historial
- `ProfileInfoRow`: Fila de información en perfil
- `RouteDetailItem`: Detalle individual (distancia, tiempo, costo)

## 🔧 Tecnologías y Versiones

### Core
- **Kotlin**: 2.0.21
- **Android Gradle Plugin**: 8.13.1
- **Min SDK**: 24 (Android 7.0)
- **Target SDK**: 36 (Android 14)
- **Compile SDK**: 36

### Jetpack Compose
- **Compose BOM**: 2024.09.00
- **Material 3**: Incluido en BOM
- **Navigation Compose**: 2.8.4
- **Activity Compose**: 1.8.0

### Firebase
- **Firebase BOM**: 33.6.0
- **Auth**: Incluido en BOM
- **Firestore**: Incluido en BOM
- **Storage**: Incluido en BOM

### Networking
- **Retrofit**: 2.9.0
- **OkHttp**: 4.12.0
- **Gson Converter**: 2.9.0

### Utilities
- **Coil**: 2.5.0 (Carga de imágenes)
- **Lifecycle ViewModel**: 2.9.4
- **Kotlin Serialization**: 1.6.0

## 📊 Modelos de Datos

### User
```kotlin
data class User(
    val id: String,
    val email: String,
    val name: String,
    val photoUrl: String,
    val createdAt: Long
)
```

### Airport
```kotlin
data class Airport(
    val id: String,
    val name: String,
    val city: String,
    val country: String,
    val iataCode: String,
    val latitude: Double,
    val longitude: Double
)
```

### Route
```kotlin
data class Route(
    val id: String,
    val userId: String,
    val origin: Airport,
    val destination: Airport,
    val routeType: RouteType,
    val distance: Double,      // km
    val estimatedTime: Double,  // horas
    val estimatedCost: Double,  // USD
    val stops: List<Airport>,
    val calculatedAt: Long
)
```

### RouteType
```kotlin
enum class RouteType {
    FASTEST,    // Minimiza tiempo
    CHEAPEST    // Minimiza costo
}
```

## 🧮 Algoritmos Implementados

### Fórmula de Haversine
Calcula distancia entre dos puntos en una esfera (la Tierra):

```kotlin
fun calculateDistance(lat1: Double, lon1: Double, lat2: Double, lon2: Double): Double {
    val R = 6371.0 // Radio de la Tierra en km
    
    val lat1Rad = Math.toRadians(lat1)
    val lat2Rad = Math.toRadians(lat2)
    val deltaLat = Math.toRadians(lat2 - lat1)
    val deltaLon = Math.toRadians(lon2 - lon1)
    
    val a = sin(deltaLat/2)² + cos(lat1Rad) * cos(lat2Rad) * sin(deltaLon/2)²
    val c = 2 * atan2(√a, √(1-a))
    
    return R * c
}
```

**Resultado**: Distancia en kilómetros con alta precisión

### Estimación de Tiempo
```kotlin
val time = if (routeType == FASTEST) {
    distance / 800.0  // km/h velocidad promedio vuelo directo
} else {
    distance / 650.0  // km/h con escalas
}
```

### Estimación de Costo
```kotlin
val cost = if (routeType == CHEAPEST) {
    distance * 0.08  // USD por km (tarifa económica)
} else {
    distance * 0.12  // USD por km (tarifa directa)
}
```

## 🔐 Seguridad

### Reglas de Firestore
```javascript
// Usuarios solo acceden a sus datos
match /users/{userId} {
  allow read, write: if request.auth.uid == userId;
}

// Rutas privadas por usuario
match /routes/{routeId} {
  allow create: if request.auth != null;
  allow read: if resource.data.userId == request.auth.uid;
}
```

### Reglas de Storage
```javascript
// Fotos de perfil privadas
match /profile_photos/{userId}.jpg {
  allow read: if request.auth != null;
  allow write: if request.auth.uid == userId;
}
```

## 🎯 Estado y Gestión

### StateFlow Pattern
```kotlin
// ViewModel
private val _uiState = MutableStateFlow(InitialState)
val uiState: StateFlow<State> = _uiState

// UI
val uiState by viewModel.uiState.collectAsStateWithLifecycle()
```

### Loading States
Cada ViewModel maneja:
- `isLoading`: Boolean para mostrar progress
- `error`: String? para mensajes de error
- `data`: Datos principales
- `isSuccess`: Boolean para operaciones exitosas

## 🔄 Navegación

### Navigation Graph
```
Login → Register ⇄
  ↓ (on success)
Home ⇄ History
  ↓
Profile → Logout → Login
```

### Deep Links (Preparado)
```kotlin
sealed class Screen(val route: String) {
    object Login : Screen("login")
    object Register : Screen("register")
    object Home : Screen("home")
    object History : Screen("history")
    object Profile : Screen("profile")
}
```

## 📱 Compatibilidad

### Dispositivos Soportados
- **Teléfonos**: Android 7.0+ (API 24+)
- **Tablets**: Diseño responsive
- **Plegables**: Soporte adaptativo

### Orientaciones
- Portrait (principal)
- Landscape (soportado)

## 🧪 Testing (Preparado)

### Estructura para Tests
```
app/src/
├── test/              # Unit tests
│   └── java/
│       └── com/example/complejidad/
│           ├── viewmodel/    # ViewModel tests
│           └── repository/   # Repository tests
└── androidTest/       # Integration tests
    └── java/
        └── com/example/complejidad/
            └── ui/          # UI tests con Compose
```

### Frameworks Configurados
- JUnit 4 para unit tests
- Espresso para UI tests
- Compose UI Test para Compose screens

## 🚀 Optimizaciones

### Performance
- LazyColumn para listas (scroll eficiente)
- Coil con caché para imágenes
- StateFlow para reactive UI
- remember y rememberSaveable para estados

### Memory
- ViewModel sobrevive a rotaciones
- Coil libera memoria automáticamente
- Firestore con caché local

## 🔌 Preparado para Backend

### Retrofit Setup Listo
```kotlin
interface RouteApiService {
    @POST("routes/calculate")
    suspend fun calculateRoute(@Body request: RouteRequest): Response<RouteResponse>
    
    @GET("airports")
    suspend fun getAirports(): Response<List<Airport>>
}
```

### Solo Falta
1. Crear implementación de RetrofitInstance
2. Actualizar RouteRepository para usar API
3. Configurar BASE_URL

## 📈 Métricas del Proyecto

- **Pantallas**: 5 (Login, Register, Home, History, Profile)
- **ViewModels**: 3 (Auth, User, Route)
- **Repositories**: 3 (Auth, User, Route)
- **Models**: 3 (User, Airport, Route)
- **Archivos Kotlin**: ~15
- **Líneas de código**: ~2,500+
- **Tiempo de desarrollo**: Completado

## 🎓 Conceptos Aplicados

✅ Grafos (representación de red de aeropuertos)  
✅ Algoritmos de caminos mínimos (preparado para Dijkstra/Bellman-Ford)  
✅ Fórmula de Haversine (distancias geodésicas)  
✅ Estructuras de datos (listas, árboles, grafos)  
✅ Complejidad algorítmica (O(n log n) para búsquedas)  
✅ Bases de datos (Firestore NoSQL)  
✅ Arquitectura de software (MVVM, Repository)  
✅ Programación reactiva (StateFlow)  

## 📝 Notas Adicionales

### Por Implementar (Futuro)
- [ ] Visualización de mapa con rutas
- [ ] Algoritmos reales Dijkstra/Bellman-Ford desde backend
- [ ] Cache offline con Room
- [ ] Notificaciones push
- [ ] Compartir rutas
- [ ] Exportar a PDF
- [ ] Dark mode automático
- [ ] Más aeropuertos (1500+ disponibles)
- [ ] Filtros avanzados
- [ ] Gráficos de análisis

### Consideraciones
- Firebase tiene límite gratuito (10GB Firestore, 1GB Storage)
- Fórmula de Haversine asume Tierra esférica (error < 0.5%)
- Costos y tiempos son estimaciones, no datos reales de aerolíneas

---

**Documentación Técnica Completa** ✅
