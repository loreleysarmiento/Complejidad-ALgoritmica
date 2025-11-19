# Guía de Integración con Backend

Esta guía te ayudará a conectar la aplicación Android con tu backend de rutas aéreas.

## 🔧 Estructura Actual

Actualmente, la aplicación usa simulación local en `RouteRepository.kt`:
- Calcula distancias con fórmula de Haversine
- Estima tiempos y costos con factores fijos
- Guarda resultados en Firebase Firestore

## 🔌 Pasos para Integrar Backend Real

### 1. Crear API Service Interface

Crea el archivo `app/src/main/java/com/example/complejidad/data/api/RouteApiService.kt`:

```kotlin
package com.example.complejidad.data.api

import retrofit2.Response
import retrofit2.http.*

interface RouteApiService {
    @POST("routes/calculate")
    suspend fun calculateRoute(
        @Body request: RouteCalculationRequest
    ): Response<RouteCalculationResponse>
    
    @GET("airports")
    suspend fun getAirports(
        @Query("country") country: String? = null
    ): Response<List<AirportDto>>
}

// DTOs (Data Transfer Objects)
data class RouteCalculationRequest(
    val originIataCode: String,
    val destinationIataCode: String,
    val algorithm: String, // "dijkstra" o "bellman-ford"
    val optimizationType: String // "fastest" o "cheapest"
)

data class RouteCalculationResponse(
    val origin: AirportDto,
    val destination: AirportDto,
    val distance: Double,
    val estimatedTime: Double,
    val estimatedCost: Double,
    val stops: List<AirportDto>,
    val path: List<String> // IATAs de los aeropuertos en la ruta
)

data class AirportDto(
    val id: String,
    val name: String,
    val city: String,
    val country: String,
    val iataCode: String,
    val latitude: Double,
    val longitude: Double
)
```

### 2. Crear Retrofit Instance

Crea el archivo `app/src/main/java/com/example/complejidad/data/api/RetrofitInstance.kt`:

```kotlin
package com.example.complejidad.data.api

import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit

object RetrofitInstance {
    // Cambia esta URL por la de tu backend
    private const val BASE_URL = "http://10.0.2.2:8080/api/" // Para emulador
    // private const val BASE_URL = "http://tu-ip:8080/api/" // Para dispositivo físico
    // private const val BASE_URL = "https://tu-backend.com/api/" // Para producción
    
    private val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    }
    
    private val okHttpClient = OkHttpClient.Builder()
        .addInterceptor(loggingInterceptor)
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        .build()
    
    val api: RouteApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(RouteApiService::class.java)
    }
}
```

### 3. Actualizar RouteRepository

Modifica `RouteRepository.kt` para usar la API:

```kotlin
package com.example.complejidad.data.repository

import com.example.complejidad.data.api.RetrofitInstance
import com.example.complejidad.data.api.RouteCalculationRequest
import com.example.complejidad.data.model.Route
import com.example.complejidad.data.model.RouteType
import com.example.complejidad.data.model.Airport
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.firestore.FirebaseFirestore
import kotlinx.coroutines.tasks.await

class RouteRepository {
    private val firestore = FirebaseFirestore.getInstance()
    private val auth = FirebaseAuth.getInstance()
    private val routesCollection = firestore.collection("routes")
    
    // Usar API real en lugar de simulación
    suspend fun calculateRoute(
        origin: Airport,
        destination: Airport,
        routeType: RouteType
    ): Result<Route> {
        return try {
            // Llamar al backend
            val request = RouteCalculationRequest(
                originIataCode = origin.iataCode,
                destinationIataCode = destination.iataCode,
                algorithm = "dijkstra", // o "bellman-ford" según prefieras
                optimizationType = if (routeType == RouteType.FASTEST) "fastest" else "cheapest"
            )
            
            val response = RetrofitInstance.api.calculateRoute(request)
            
            if (response.isSuccessful && response.body() != null) {
                val data = response.body()!!
                
                val route = Route(
                    userId = auth.currentUser?.uid ?: "",
                    origin = origin,
                    destination = destination,
                    routeType = routeType,
                    distance = data.distance,
                    estimatedTime = data.estimatedTime,
                    estimatedCost = data.estimatedCost,
                    stops = data.stops.map { dto ->
                        Airport(
                            id = dto.id,
                            name = dto.name,
                            city = dto.city,
                            country = dto.country,
                            iataCode = dto.iataCode,
                            latitude = dto.latitude,
                            longitude = dto.longitude
                        )
                    }
                )
                
                // Guardar en Firestore
                val docRef = routesCollection.add(route).await()
                Result.success(route.copy(id = docRef.id))
            } else {
                Result.failure(Exception("Error: ${response.code()} - ${response.message()}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    // Resto de funciones sin cambios...
}
```

### 4. Añadir Dependencia de Logging (Opcional pero recomendado)

En `gradle/libs.versions.toml`:

```toml
[versions]
# ... otros ...
okhttp = "4.12.0"

[libraries]
# ... otros ...
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
```

En `app/build.gradle.kts`:

```kotlin
dependencies {
    // ... otros ...
    implementation(libs.okhttp.logging)
}
```

## 📡 Endpoints Esperados del Backend

### 1. POST /api/routes/calculate

**Request:**
```json
{
  "originIataCode": "LIM",
  "destinationIataCode": "EZE",
  "algorithm": "dijkstra",
  "optimizationType": "fastest"
}
```

**Response (200 OK):**
```json
{
  "origin": {
    "id": "2564",
    "name": "Aeropuerto Internacional Jorge Chávez",
    "city": "Lima",
    "country": "Perú",
    "iataCode": "LIM",
    "latitude": -12.0219,
    "longitude": -77.1143
  },
  "destination": {
    "id": "2610",
    "name": "Aeropuerto Internacional Ministro Pistarini",
    "city": "Buenos Aires",
    "country": "Argentina",
    "iataCode": "EZE",
    "latitude": -34.8222,
    "longitude": -58.5358
  },
  "distance": 3121.45,
  "estimatedTime": 3.9,
  "estimatedCost": 374.57,
  "stops": [
    // Lista de aeropuertos intermedios (si los hay)
  ],
  "path": ["LIM", "SCL", "EZE"]
}
```

### 2. GET /api/airports

**Request:**
```
GET /api/airports?country=Peru
```

**Response (200 OK):**
```json
[
  {
    "id": "2564",
    "name": "Aeropuerto Internacional Jorge Chávez",
    "city": "Lima",
    "country": "Perú",
    "iataCode": "LIM",
    "latitude": -12.0219,
    "longitude": -77.1143
  },
  // ... más aeropuertos
]
```

## 🔧 Configuración para Testing

### Emulador Android
- Usa `http://10.0.2.2:8080/api/` para acceder a localhost de tu PC

### Dispositivo Físico
1. Conecta tu PC y dispositivo a la misma red WiFi
2. Obtén la IP de tu PC: `ipconfig` (Windows) o `ifconfig` (Mac/Linux)
3. Usa `http://TU_IP:8080/api/`

### Producción
- Despliega tu backend en un servidor
- Actualiza BASE_URL a la URL de producción
- Considera usar HTTPS

## 🧪 Testing de la API

Puedes probar tu backend con Postman o curl:

```bash
# Calcular ruta
curl -X POST http://localhost:8080/api/routes/calculate \
  -H "Content-Type: application/json" \
  -d '{
    "originIataCode": "LIM",
    "destinationIataCode": "EZE",
    "algorithm": "dijkstra",
    "optimizationType": "fastest"
  }'

# Obtener aeropuertos
curl http://localhost:8080/api/airports
```

## 🔒 Seguridad

Para producción, considera:

1. **Autenticación**: Añadir token JWT en headers
```kotlin
private val okHttpClient = OkHttpClient.Builder()
    .addInterceptor { chain ->
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        chain.proceed(request)
    }
    .build()
```

2. **HTTPS**: Usar certificados SSL/TLS

3. **Rate Limiting**: Implementar límites de peticiones

## 🐛 Troubleshooting

### Error: "Unable to resolve host"
- Verifica que BASE_URL sea correcta
- Asegura que el backend esté corriendo
- Verifica permisos de INTERNET en AndroidManifest

### Error: "Connection timeout"
- Aumenta los timeouts en OkHttpClient
- Verifica firewall/antivirus

### Error: "cleartext traffic not permitted"
- Añade `android:usesCleartextTraffic="true"` en AndroidManifest
- O usa HTTPS

## 📊 Monitoreo

Agrega logging para debug:

```kotlin
private val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = if (BuildConfig.DEBUG) {
        HttpLoggingInterceptor.Level.BODY
    } else {
        HttpLoggingInterceptor.Level.NONE
    }
}
```

## ✅ Checklist de Integración

- [ ] Backend corriendo y accesible
- [ ] Dependencias de Retrofit configuradas
- [ ] API Service interface creada
- [ ] RetrofitInstance configurado con BASE_URL correcta
- [ ] RouteRepository actualizado para usar API
- [ ] Permisos de INTERNET en AndroidManifest
- [ ] Testing con Postman/curl
- [ ] Testing en app Android
- [ ] Manejo de errores implementado
- [ ] Loading states actualizados

---

Una vez completada la integración, la app usará algoritmos reales de optimización de rutas desde tu backend!
