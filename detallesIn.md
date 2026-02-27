## Estadísticas Clave que para Recopilar

### 1. **Uso de CPU y Energía**
- **Battery Manager API** (`BatteryManager`): Obtener información del consumo de batería
- **Power Profile Data**: Consumo de energía por componente (CPU, GPU, pantalla, radio)
- **CPU Time Tracking**: Usar `android.os.Debug` para medir tiempo de CPU

```kotlin
// Ejemplo básico de lectura de batería
val batteryManager = context.getSystemService(Context. BATTERY_SERVICE) as BatteryManager
val remainingEnergy = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_ENERGY_COUNTER)
```

### 2. **Uso de Red (Datos Móviles y WiFi)**
- **TrafficStats API**: Monitorear bytes transmitidos/recibidos
- **ConnectivityManager**: Detectar tipo de conexión (4G, 5G, WiFi)
- **Radio State Tracking**: Cuando se activa/desactiva la radio

```kotlin
// Ejemplo de TrafficStats
val rxBytes = TrafficStats.getTotalRxBytes()
val txBytes = TrafficStats.getTotalTxBytes()
```

### 3. **Uso de GPS y Localización**
- **LocationManager**: Frecuencia y duración del uso de GPS
- **Accuracy levels**: GPS de alta precisión consume más energía
- **Update intervals**: Rastrear frecuencia de actualizaciones

### 4. **Uso de Pantalla**
- **Display metrics**: Brillo, tiempo de encendido
- **Screen State**: Mediante `BroadcastReceiver` para detectar cambios
- **Display technology**:  OLED vs LCD tienen diferentes consumos

### 5. **Uso de Sensores**
- **Acelerómetro, giróscopo, brújula**: Consumo de energía variable
- **Sensor manager events**: Frecuencia de recolección de datos

### 6. **Procesos en Background**
- **WorkManager**:  Tareas programadas
- **Foreground Services**: Detectar qué apps ejecutan servicios
- **Wake locks**: Apps que mantienen el dispositivo despierto

## Arquitectura Recomendada

```
┌─────────────────────────────────────┐
│     Jetpack Compose UI              │
│  (Dashboard de estadísticas)        │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│   ViewModel / MVVM                  │
│  (Lógica de presentación)          │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│   Repository Pattern                │
│  (Abstracción de datos)            │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│   Data Layer:                        │
│  • Battery Manager API              │
│  • TrafficStats                     │
│  • LocationManager                  │
│  • SensorManager                    │
│  • Power Usage Statistics           │
│  • Local Database (Room)            │
└─────────────────────────────────────┘
```

## Permisos Necesarios (AndroidManifest.xml)

```xml
<!-- Lectura de estadísticas de batería y energía -->
<uses-permission android:name="android.permission. BATTERY_STATS" />

<!-- Acceso a la red -->
<uses-permission android:name="android. permission.ACCESS_NETWORK_STATE" />

<!-- Localización -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android. permission.ACCESS_COARSE_LOCATION" />

<!-- Sensores -->
<uses-permission android:name="android.permission. BODY_SENSORS" />

<!-- Lectura de paquetes instalados -->
<uses-permission android:name="android.permission. QUERY_ALL_PACKAGES" />

<!-- Para monitoreo en background -->
<uses-permission android:name="android.permission. PACKAGE_USAGE_STATS" />
```

## Pasos de Implementación

### **Fase 1: Recopilación Básica**
1. Crear servicio que recopile métricas principales
2. Almacenar datos en Room Database
3. Crear dashboard simple en Compose

### **Fase 2: Análisis y Cálculo de Huella**
1. Mapear consumo de energía a emisiones de carbono
2. Usar modelos de consumo de red (bits ≈ CO2)
3. Análisis por aplicación/tipo de actividad

### **Fase 3: Visualización Avanzada**
1. Gráficos de tendencias temporales
2. Comparativas entre apps
3. Recomendaciones de optimización

## Fuentes de Datos de Carbono

Para mapear energía → CO2:
- **EPA Carbon Intensity Database**:  Emisiones por región (grid eléctrico)
- **Network Carbon API**: Huella de carbono de transmisión de datos
- **Device Power Profiles**: Consumo específico por modelo de teléfono

## Desafíos Principales

⚠️ **Limitaciones en Android 12+**:  Acceso restringido a estadísticas detalladas
⚠️ **Exactitud**: El consumo real varía según hardware y carga del sistema
⚠️ **Background restrictions**: Difícil monitorear en tiempo real en versiones recientes
⚠️ **Privacidad**: Necesitas ser cuidadoso con qué datos recopilas

---

## Análisis por Tipo de App

### **1. Instagram / TikTok (Apps de Redes Sociales / Video)**

#### Consumo Principal: 
```
📱 PANTALLA (40-50% del consumo)
  ├─ Scroll continuo (siempre encendida)
  ├─ Reproducción de videos (variable según resolución)
  └─ Transiciones y animaciones de UI

📡 RED (30-40% del consumo)
  ├─ Descarga de imágenes/videos (high bandwidth)
  ├─ Streaming de video (H.264/VP9 codecs)
  ├─ Upload de contenido (si el usuario publica)
  └─ Actualizaciones en tiempo real del feed

🎬 GPU/CPU (15-25%)
  ├─ Decodificación de video
  ├─ Procesamiento de imagen (filtros)
  └─ Renderizado de UI compleja

📍 SENSORES (5-10%)
  ├─ Acelerómetro (para reels/stories)
  ├─ Micrófono (si graba videos)
  └─ Cámara (captura de fotos/videos)
```

### **2. WhatsApp (Mensajería)**

#### Consumo Principal: 
```
📡 RED (40-50%)
  ├─ Sincronización de mensajes (pequeño volumen pero frecuente)
  ├─ Transmisión de voice notes (comprimidas con Opus codec)
  ├─ Transmisión de video calls (H.264)
  └─ Descarga de archivos/imágenes

📱 PANTALLA (30-40%)
  ├─ Pantalla encendida durante chats
  ├─ Notificaciones que activan pantalla
  └─ Menor carga que redes sociales

🎤🎵 AUDIO (10-15%)
  ├─ Codificación de voice notes (Opus)
  ├─ Reproducción de audio
  └─ Video calls (usa mucho si es frecuente)

📍 LOCALIZACIÓN (5%)
  ├─ Compartir ubicación (si está activo)
  └─ Ubicación de contactos cercanos
```

## Cómo Medir Estos Datos en Android

### **A) Rastrear Consumo por Aplicación (Package Level)**

```kotlin
// Obtener estadísticas de batería por app
fun getAppBatteryUsage(context: Context, packageName: String): Long {
    val batteryManager = context.getSystemService(Context.BATTERY_SERVICE) as BatteryManager
    
    // Para Android 8+, necesitas UsageStatsManager
    val usageStatsManager = context.getSystemService(Context.USAGE_STATS_SERVICE) 
        as UsageStatsManager
    
    val currentTime = System.currentTimeMillis()
    val oneHourAgo = currentTime - (60 * 60 * 1000)
    
    val queryUsageStats = usageStatsManager.queryUsageStats(
        UsageStatsManager.INTERVAL_HOURLY,
        oneHourAgo,
        currentTime
    )
    
    queryUsageStats.forEach { stat ->
        if (stat.packageName == packageName) {
            println("App: ${stat.packageName}")
            println("Tiempo en foreground: ${stat.totalTimeInForeground}")
            println("Últimos:  ${stat.lastTimeUsed}")
        }
    }
}
```

### **B) Rastrear Tráfico de Red por App**

```kotlin
// TrafficStats NO permite filtrar por app directamente
// Necesitas usar eBPF o monitoreo a nivel de sistema
fun getNetworkStatsPerApp(context: Context, packageName: String) {
    val networkStatsManager = context.getSystemService(Context.NETWORK_STATS_SERVICE) 
        as NetworkStatsManager
    
    val currentTime = System.currentTimeMillis()
    val oneHourAgo = currentTime - (60 * 60 * 1000)
    
    try {
        // Android 6+
        val mobileStats = networkStatsManager.queryDetailsForDevice(
            ConnectivityManager.TYPE_MOBILE,
            "",
            oneHourAgo,
            currentTime
        )
        
        mobileStats.use { bucket ->
            while (bucket.hasNextBucket()) {
                bucket.getNextBucket()
                val uid = bucket.uid
                val rxBytes = bucket.rxBytes
                val txBytes = bucket.txBytes
                
                // Mapear UID a packageName
                println("RX: $rxBytes, TX: $txBytes")
            }
        }
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```

### **C) Detectar Actividades Específicas**

```kotlin
// Detectar si Instagram está reproduciendo video
fun monitorDisplayState(context: Context) {
    val displayManager = context.getSystemService(Context. DISPLAY_SERVICE) as DisplayManager
    
    displayManager.displays.forEach { display ->
        println("Display mode: ${display.mode}")
        println("Refresh rate: ${display.mode?. refreshRate}")
        println("Resolution: ${display.mode?.physicalWidth} x ${display.mode?.physicalHeight}")
    }
}

// Detectar transmisión de video/audio
fun monitorAudioFocus(context: Context) {
    val audioManager = context.getSystemService(Context.AUDIO_SERVICE) as AudioManager
    
    // Escuchar cambios en el foco de audio
    audioManager.requestAudioFocus(
        AudioManager.AUDIOFOCUS_GAIN,
        AudioManager.STREAM_MUSIC
    )
}

// Detectar uso de cámara
fun monitorCameraUsage(context: Context) {
    val cameraManager = context.getSystemService(Context.CAMERA_SERVICE) as CameraManager
    
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.Q) {
        cameraManager.registerAvailabilityCallback(
            object : CameraManager.AvailabilityCallback() {
                override fun onCameraAccessPrioritiesChanged() {
                    super.onCameraAccessPrioritiesChanged()
                    println("Camera access changed")
                }
            }
        )
    }
}
```

## Modelo de Consumo de Energía

### **Para Instagram/TikTok:**
```
Energía Total = 
    (Tiempo pantalla × Brillo × Tipo LCD/OLED) +
    (Datos descargados × Consumo red 4G/5G/WiFi) +
    (Datos cargados × Consumo red) +
    (Tiempo decodificación video × CPU power) +
    (Tiempo GPU × GPU power) +
    (Activaciones sensor × Consumo sensor)
```

**Valores típicos:**
- 4G: ~0.6 mAh por MB
- 5G: ~0.4 mAh por MB (más eficiente)
- WiFi: ~0.1 mAh por MB (mucho más eficiente)
- Pantalla OLED: 2-3 mA/min (brillo medio)
- Pantalla LCD: 1-2 mA/min (brillo medio)
- Video 1080p @ 30fps: 500-800 mAh/hora

### **Para WhatsApp:**
```
Energía Total =
    (Tiempo en foreground × Consumo base) +
    (Mensajes enviados/recibidos × Overhead red) +
    (Voice notes × Duración × Consumo codec) +
    (Video calls × Duración × Consumo H.264) +
    (Tiempo compartir ubicación × Consumo GPS)
```

**Valores típicos:**
- WhatsApp idle: ~10 mAh/hora
- WhatsApp activo (textos): ~30 mAh/hora
- WhatsApp voice call: ~200-300 mAh/hora
- WhatsApp video call: ~400-600 mAh/hora

## Estructura de Base de Datos (Room)

```kotlin
@Entity(tableName = "app_metrics")
data class AppMetric(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val packageName: String,
    val appName: String,
    val timestamp: Long,
    
    // Energía
    val batteryUsageEstimate: Float,  // mAh
    
    // Red
    val dataDownloaded: Long,  // bytes
    val dataUploaded:  Long,    // bytes
    val networkType: String,   // "4G", "5G", "WiFi"
    
    // Pantalla
    val screenTimeMs: Long,
    val brightnessLevel: Int,  // 0-255
    
    // CPU/GPU
    val cpuUsagePercent: Float,
    val gpuUsagePercent: Float,
    
    // Actividad
    val isVideoPlayback: Boolean,
    val isAudioPlayback: Boolean,
    val isCameraActive: Boolean,
    val isGpsActive: Boolean,
    
    // Huella de Carbono
    val estimatedCo2Grams: Float
)

@Dao
interface AppMetricDao {
    @Insert
    suspend fun insert(metric: AppMetric)
    
    @Query("SELECT * FROM app_metrics WHERE packageName = :packageName ORDER BY timestamp DESC")
    fun getMetricsForApp(packageName: String): Flow<List<AppMetric>>
}
```

## Permisos Adicionales Necesarios

```xml
<!-- Para obtener estadísticas detalladas -->
<uses-permission android:name="android.permission. PACKAGE_USAGE_STATS" />

<!-- Para acceso a red detallado -->
<uses-permission android: name="android.permission.ACCESS_NETWORK_STATE" />

<!-- Para detectar instalación de apps -->
<uses-permission android:name="android.permission. QUERY_ALL_PACKAGES" />

<!-- Para monitoreo de sensores -->
<uses-permission android:name="android.permission. BODY_SENSORS" />

<!-- Para cámara y micrófono (si quieres detectar cuando está activa) -->
<uses-permission android:name="android.permission. CAMERA" />
<uses-permission android:name="android.permission. RECORD_AUDIO" />
```