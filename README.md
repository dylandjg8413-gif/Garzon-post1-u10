SecureMobile — Pipeline DevSecOps Android con GitLab CI/CD

Unidad 10: Integración Continua, Seguridad y Despliegue — Post-Contenido 1 Ingeniería de Sistemas · 2026

Descripción

Este proyecto implementa un pipeline DevSecOps completo para una aplicación Android utilizando GitLab CI/CD. El pipeline automatiza las siguientes etapas:

Static Analysis — Inspección de calidad y seguridad con Detekt y Android Lint
Unit Testing — Ejecución automatizada de pruebas unitarias con JUnit
Security Scan — Escaneo de dependencias vulnerables con OWASP Dependency Check
Coverage Gate — Validación de cobertura mínima ≥ 70% usando JaCoCo
Signed Build — Generación automática del APK firmado
Deployment — Publicación automática en Firebase App Distribution

🔄 Flujo del Pipeline
┌─────────────┐     ┌──────────────────┐     ┌────────────────────┐
│ Merge /Push │────▶│ quality-security │────▶│ deploy-release     │
│ a develop   │     │                  │     │ (solo en release)  │
└─────────────┘     │ ✓ Lint           │     │                    │
                    │ ✓ Detekt         │     │ ✓ Decode Keystore  │
                    │ ✓ Unit Tests     │     │ ✓ Build Signed APK │
                    │ ✓ Coverage Gate  │     │ ✓ Firebase Deploy  │
                    │ ✓ OWASP Scan     │     │                    │
                    └──────────────────┘     └────────────────────┘
Diagrama de etapas
Developer ──push──▶ GitLab Repository ──trigger──▶ GitLab CI/CD
                                                      │
                                      ┌───────────────┴──────────────┐
                                      ▼                              ▼
                              quality-security              deploy-release
                              ┌──────────────┐             ┌────────────────┐
                              │ 1. Lint      │             │ 5. Decode      │
                              │ 2. Detekt    │────▶        │    Keystore    │
                              │ 3. Unit Test │ needs       │ 6. Build APK   │
                              │ 4. JaCoCo    │             │    Signed      │
                              │    ≥ 70%     │             │ 7. Firebase    │
                              │ OWASP Check  │             │    Distribution│
                              └──────────────┘             └────────────────┘
                                                               │
                                                               ▼
                                                        QA Team recibe
                                                        nueva versión

Configuración de Variables Seguras

Para que el pipeline funcione correctamente, se deben configurar las siguientes Variables CI/CD en:

Settings → CI/CD → Variables

Variable	Descripción	Cómo obtenerla
KEYSTORE_BASE64	Keystore codificado en Base64	base64 release-key.jks
KEYSTORE_PASSWORD	Contraseña del keystore	Definida con keytool
KEY_ALIAS	Alias de la clave	Configurado al generar el keystore
KEY_PASSWORD	Contraseña del alias	Definida al crear la clave
FIREBASE_APP_ID	ID de aplicación Firebase	Firebase Console
FIREBASE_TOKEN	Token Firebase CLI	firebase login:ci
Generar el Keystore
keytool -genkeypair \
  -v \
  -keystore secure-release.jks \
  -alias secureapp \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
Codificar el Keystore en Base64

# Windows PowerShell

[Convert]::ToBase64String([IO.File]::ReadAllBytes("secure-release.jks"))

# Linux / macOS
base64 secure-release.jks

Configuración de Firma Segura

El bloque signingConfigs dentro de app/build.gradle.kts obtiene automáticamente las credenciales desde las variables de entorno configuradas en GitLab CI/CD:

signingConfigs {
    create("release") {
        storeFile = file(System.getenv("KEYSTORE_PATH") ?: "secure-release.jks")
        storePassword = System.getenv("KEYSTORE_PASSWORD") ?: ""
        keyAlias = System.getenv("KEY_ALIAS") ?: ""
        keyPassword = System.getenv("KEY_PASSWORD") ?: ""
    }
}

Importante:
El archivo .jks nunca debe almacenarse en el repositorio. Se encuentra excluido mediante .gitignore.

Coverage Gate — JaCoCo

Se configuró un umbral mínimo de cobertura del 70% utilizando JaCoCo.
El pipeline se detiene automáticamente si la cobertura baja de este porcentaje.

Herramienta: JaCoCo 0.8.11
Reportes generados: HTML y XML
Ubicación: app/build/reports/jacoco/
Verificación automática desde GitLab CI/CD
Escaneo de Seguridad — OWASP Dependency Check

El proyecto incorpora análisis de vulnerabilidades para detectar dependencias inseguras o desactualizadas.

Características:

Escaneo automático de librerías Gradle
Detección de vulnerabilidades CVE
Generación de reportes HTML
Integración automática en el pipeline

Ruta del reporte:

app/build/reports/dependency-check-report.html
Verificar Firma del APK

Para comprobar que el APK se encuentra correctamente firmado:

apksigner verify --verbose app/build/outputs/apk/release/app-release.apk

La salida debe indicar:

Verified using v1 scheme
Verified using v2 scheme
Estructura del Proyecto
├── .gitlab-ci.yml                  # Pipeline DevSecOps
├── app/
│   ├── build.gradle.kts            # Configuración Android
│   ├── proguard-rules.pro
│   └── src/
│       ├── main/
│       │   ├── AndroidManifest.xml
│       │   └── java/.../
│       │       ├── LoginActivity.kt
│       │       └── AuthManager.kt
│       └── test/
│           └── java/.../
│               └── AuthManagerTest.kt
├── gradle/
├── gradlew
├── gradlew.bat
├── settings.gradle.kts
├── .gitignore
└── README.md

Checkpoints
Checkpoint 1: Pipeline de Calidad y Seguridad
El archivo .gitlab-ci.yml fue agregado correctamente
Las variables CI/CD fueron configuradas en GitLab
El stage quality-security finaliza exitosamente
Los reportes de pruebas y seguridad se generan correctamente
El escaneo OWASP no detecta vulnerabilidades críticas

Evidencia — Calidad y Seguridad

Checkpoint 1 — Lint, Testing y OWASP exitosos

Checkpoint 2: Build Firmado y Distribución
El stage deploy-release solo se ejecuta en rama release
El APK generado está firmado correctamente
Firebase App Distribution publica automáticamente el build
El equipo QA recibe la notificación de despliegue
Evidencia — Distribución Continua

Checkpoint 2 — APK firmado y desplegado

Checkpoint 3: Coverage Gate y Reportes
JaCoCo genera reportes HTML y XML
GitLab almacena artefactos automáticamente
El pipeline falla si la cobertura es menor al 70%
Los reportes pueden descargarse desde GitLab CI/CD
Evidencia — Cobertura y Artefactos

Checkpoint 3 — Reportes y Coverage Gate

Cómo Ejecutar Localmente
# Compilar aplicación
./gradlew assembleDebug

# Ejecutar pruebas unitarias
./gradlew testDebugUnitTest

# Generar reporte JaCoCo
./gradlew jacocoTestReport

# Verificar cobertura mínima
./gradlew jacocoCoverageVerification

# Ejecutar Android Lint
./gradlew lintDebug

# Ejecutar análisis Detekt
./gradlew detekt

# Ejecutar análisis OWASP
./gradlew dependencyCheckAnalyze
