# 🚀 Quick Start Guide - SkyRoute

## ⚡ Inicio Rápido (5 minutos)

### 1️⃣ Configurar Firebase

1. **Ir a Firebase Console**: https://console.firebase.google.com/
2. **Crear proyecto**: Click en "Agregar proyecto" → Nombra "SkyRoute"
3. **Agregar app Android**:
   - Click en ícono Android
   - Package name: `com.example.complejidad`
   - Descargar `google-services.json`
   - **Reemplazar** el archivo en `app/google-services.json`

4. **Habilitar servicios** (3 clics):
   - **Authentication** → Get Started → Email/Password → Enable
   - **Firestore Database** → Create Database → Production mode → Next → Enable
   - **Storage** → Get Started → Production mode → Done

### 2️⃣ Ejecutar la App

```bash
# En Android Studio:
1. File → Sync Project with Gradle Files
2. Click Run ▶️
```

### 3️⃣ Probar

1. **Registrarse**: 
   - Click "Regístrate"
   - Nombre: Tu nombre
   - Email: tu@email.com
   - Contraseña: mínimo 6 caracteres

2. **Calcular Ruta**:
   - Origen: Lima (LIM)
   - Destino: Buenos Aires (EZE)
   - Tipo: Más Rápida
   - Click "Calcular Ruta"

3. **Ver Historial**: Click ícono reloj ⏱️

4. **Ver Perfil**: Click ícono persona 👤

## ✅ ¡Listo! Tu app está funcionando

---

## 📱 Funcionalidades Implementadas

✅ Login y Registro  
✅ Búsqueda de rutas (origen/destino)  
✅ Cálculo con fórmula de Haversine  
✅ Dos tipos: Rápida / Económica  
✅ Historial de rutas  
✅ Perfil de usuario  
✅ Foto de perfil (cambiar desde galería)  
✅ Cerrar sesión  
✅ Diseño moderno con tema azul aviación  

## 🔧 Próximo Paso: Conectar Backend

Cuando tu backend esté listo:
1. Lee `BACKEND_INTEGRATION.md`
2. Actualiza `BASE_URL` en código
3. La app usará Dijkstra/Bellman-Ford real

## 📚 Documentación Completa

- `README.md` - Documentación general
- `PROJECT_SUMMARY.md` - Resumen completo del proyecto
- `BACKEND_INTEGRATION.md` - Guía para integrar backend

## 🎯 Aeropuertos Disponibles

- Lima (LIM) 🇵🇪
- Bogotá (BOG) 🇨🇴
- Buenos Aires (EZE) 🇦🇷
- São Paulo (GRU) 🇧🇷
- Santiago (SCL) 🇨🇱
- Quito (UIO) 🇪🇨

## ⚠️ Importante

- Sin Firebase configurado → App no funciona
- Necesitas internet para usar la app
- El `google-services.json` incluido es placeholder

---

**¿Problemas?** Revisa README.md sección "Solución de Problemas"
