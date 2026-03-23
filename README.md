# 🤖 AI Device Optimizer

Android-приложение с AI-оптимизатором устройства через **Shizuku**.  
Стек: **Kotlin + Jetpack Compose**, архитектура **MVVM + Clean**.

---

## 📁 Структура проекта

```
app/src/main/
├── aidl/com/aioptimizer/
│   └── IShizukuService.aidl          # AIDL интерфейс для IPC с Shizuku
│
├── kotlin/com/aioptimizer/
│   ├── AIOptimizerApp.kt             # Application класс
│   │
│   ├── service/
│   │   ├── ShizukuUserService.kt     # 🔑 Привилегированный сервис (shell-уровень)
│   │   └── OptimizerForegroundService.kt  # Foreground Service + BootReceiver
│   │
│   ├── helper/
│   │   └── ShizukuHelper.kt          # Управление соединением с Shizuku
│   │
│   ├── optimizer/
│   │   ├── OptimizationMode.kt       # 4 режима + параметры
│   │   └── AIOptimizer.kt            # AI-логика анализа и оптимизации
│   │
│   ├── viewmodel/
│   │   └── MainViewModel.kt          # MVVM ViewModel
│   │
│   └── ui/
│       ├── MainActivity.kt
│       ├── theme/
│       │   ├── Theme.kt              # Тёмная тема + цвета
│       │   └── Typography.kt
│       └── screens/
│           └── HomeScreen.kt         # Весь UI на Compose
│
└── AndroidManifest.xml
```

---

## ⚙️ Как собрать

### 1. Требования
- Android Studio Hedgehog или новее
- Android SDK 35
- Устройство с Android 9+ (API 28+)
- **Shizuku** установлен и запущен на устройстве

### 2. Клонирование и открытие
```bash
# Открыть папку AIOptimizer в Android Studio как проект
File → Open → выбрать папку AIOptimizer
```

### 3. Sync и Build
```
File → Sync Project with Gradle Files
Build → Make Project
```

### 4. Установка на устройство
```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

---

## 🔑 Настройка Shizuku

### Вариант A — через ADB (без root)
```bash
# Запустить на ПК (USB debugging должен быть включён)
adb shell sh /storage/emulated/0/Android/data/moe.shizuku.privileged.api/start.sh
```

### Вариант B — через Wireless ADB (Android 11+)
1. Настройки → Система → Разработчик → Беспроводная отладка
2. В Shizuku выбрать "Запустить через Беспроводную отладку"

### Вариант C — с root
Shizuku автоматически запускается через Magisk/KernelSU.

---

## 🎮 Режимы оптимизации

| Режим | CPU | Анимации | Сеть | Батарея |
|-------|-----|----------|------|---------|
| 🎮 Игровой | performance | 0.5x | WiFi вкл | Активная |
| ⚡ Производительность | schedutil | 0.8x | Всё вкл | Нормальная |
| 🔋 Экономия | powersave | 0.5x | WiFi only | Агрессивная |
| 🌙 Ночной | powersave | 0.0x | Всё выкл | Максимальная |

---

## 🧠 AI-логика рекомендации

```
Батарея ≤ 15% && не заряжается  →  🔋 Экономия
RAM ≥ 85% занято                →  ⚡ Производительность  
CPU temp ≥ 65°C                 →  🔋 Экономия
Заряжается && WiFi               →  🎮 Игровой
По умолчанию                    →  ⚡ Производительность
```

---

## 🔧 Что делает ShizukuUserService

Запускается в shell-контексте и выполняет:

| Операция | Команда |
|----------|---------|
| CPU governor | `echo X > /sys/devices/system/cpu/cpuN/cpufreq/scaling_governor` |
| GPU governor | `echo X > /sys/class/kgsl/kgsl-3d0/devfreq/governor` |
| Kill apps | `am force-stop <package>` |
| WiFi вкл/выкл | `svc wifi enable/disable` |
| Данные | `svc data enable/disable` |
| Яркость | `settings put system screen_brightness N` |
| Анимации | `settings put global window_animation_scale N` |
| Doze | `dumpsys deviceidle enable/force-idle` |
| DNS | `ndc resolver setnetdns ...` |
| Очистка RAM | `echo 3 > /proc/sys/vm/drop_caches` |

---

## 📦 Зависимости

```kotlin
// Shizuku
"dev.rikka.shizuku:api:13.1.5"
"dev.rikka.shizuku:provider:13.1.5"

// Compose BOM 2024.12.01
// Material3, Material Icons Extended
// Navigation Compose
// Lifecycle ViewModel Compose
// DataStore Preferences
// Coroutines Android
```

---

## ⚠️ Важные замечания

1. **AIDL**: После изменения `IShizukuService.aidl` — пересоберите проект (`Build → Rebuild Project`)
2. **ProGuard**: `ShizukuUserService` должен быть в keep-правилах (уже добавлено)
3. **CPU paths**: На разных устройствах пути sysfs могут отличаться. Проверьте `/sys/devices/system/cpu/`
4. **GPU governor**: Mali — `/sys/bus/platform/drivers/mali/`, Adreno — `/sys/class/kgsl/kgsl-3d0/devfreq/`
5. **drop_caches**: Требует root или повышенных прав — через Shizuku работает в shell-контексте
