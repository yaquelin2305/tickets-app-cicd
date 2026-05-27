# 🎟️ Microservicio de Tickets - Capa DevOps Avanzada

Este repositorio contiene el **Microservicio de Tickets** desarrollado en Java 17 con Spring Boot bajo una Arquitectura Hexagonal. En esta segunda etapa del proyecto, se ha implementado un ciclo completo de **Integración Continua (CI)** y **Entrega Continua (CD)** automatizado a través de **GitHub Actions**, incorporando prácticas avanzadas de contenerización, análisis estático de código y gobernanza de seguridad.

---

## 🚀 1. Arquitectura del Pipeline de CI/CD

El flujo automatizado se encuentra configurado en el archivo `.github/workflows/ci-cd.yml` y se activa de manera automática ante cada evento de `push` en la rama principal (`main`). 

El pipeline está diseñado de forma secuencial mediante el uso de dependencias (`needs`), garantizando que la aplicación solo avance si cumple estrictamente con las políticas de calidad y seguridad establecidas.

### 📊 Diagrama del Flujo Automatizado
[ Push en Main ]
│
▼
┌──────────────┐
│    Stage 1   │
│ Test & Sonar │ ──► (Si las Pruebas o el Quality Gate fallan, el flujo SE BLOQUEA)
└──────────────┘
│ (Success)
▼
┌──────────────┐
│    Stage 2   │
│ Build Docker │ ──► (Genera la Imagen Multi-stage y la taguea con el SHA de Git)
└──────────────┘
│ (Success)
▼
┌──────────────┐
│    Stage 3   │
│ Deploy App   │ ──► (Orquesta el entorno simulado levantando la app y PostgreSQL)
└──────────────┘

## 🛠️ 2. Descripción de las Etapas del Pipeline

### 🧪 Etapa 1: Análisis de Calidad y Pruebas Unitarias (`sonar`)
* **Propósito:** Validar la estabilidad del código y asegurar que no existan regresiones.
* **Mecanismo:** Se configuran las dependencias del entorno virtual usando Java 17 (Temurin). Se ejecutan de manera automatizada las pruebas unitarias a través del comando `mvn verify sonar:sonar`.
* **Gobernanza y Bloqueo:** Los resultados son enviados directamente a **SonarCloud** (SonarQube en la nube). El pipeline está configurado de manera **bloqueante**; si las pruebas fallan o si el código no supera los umbrales de seguridad mínimos exigidos por la plataforma, el flujo se interrumpe inmediatamente para proteger la estabilidad del software.

### 🐳 Etapa 2: Contenerización y Empaquetado (`build-image`)
* **Propósito:** Aislar la aplicación y sus dependencias para garantizar su portabilidad en la nube.
* **Mecanismo:** Utiliza un archivo `Dockerfile` basado en una estrategia de **Construcción Multi-etapa (Multi-stage build)**:
  1. *Etapa de Compilación:* Usa una imagen de Maven para empaquetar el código fuente y compilar el archivo `.jar`.
  2. *Etapa de Ejecución:* Migra únicamente el artefacto final a una imagen base ligera de Java JRE basada en Alpine Linux, optimizando el tamaño y eliminando herramientas innecesarias para reducir la superficie de ataques.
* **Publicación:** La imagen resultante es almacenada de forma segura en **GitHub Packages** (`ghcr.io`).

### 📦 Etapa 3: Orquestación y Despliegue Simulado (`deploy`)
* **Propósito:** Desplegar de forma automatizada y controlada la solución tecnológica completa.
* **Mecanismo:** Mediante **Docker Compose**, el pipeline simula un entorno productivo en la nube levantando simultáneamente dos contenedores interconectados en una red aislada:
  1. `tickets_app`: El microservicio contenedorizado expuesto en el puerto `8081`.
  2. `tickets_db`: Un motor de base de datos PostgreSQL (`postgres:15-alpine`) requerido para la persistencia.
* **Estabilidad y Escalabilidad:** El archivo `docker-compose.yml` incorpora límites estrictos de recursos de hardware (restricciones de CPU y Memoria RAM por contenedor) y políticas de reinicio ante fallos (`restart_policy`), asegurando la resiliencia de la infraestructura.

---

## 🔍 3. Estrategia de Trazabilidad Total

Para cumplir con las máximas exigencias de gobernanza, el pipeline implementa un mecanismo de **Trazabilidad de Extremo a Extremo** mediante el uso del ID único del commit de Git (`${{ github.sha }}`):

1. Cada vez que se construye la imagen Docker en GitHub Actions, esta es etiquetada (tagueada) con el código alfanumérico exacto del commit que originó el cambio.
2. Durante la etapa de despliegue, el pipeline inyecta esta etiqueta en el entorno.
3. Esto permite a los administradores de sistemas y equipos de QA rastrear cualquier contenedor en ejecución en la nube y saber con absoluta certeza física qué línea de código exacta, qué autor y qué cambios originaron la versión que está corriendo en producción.

---

## ⚙️ 4. Requisitos para la Ejecución Local

Si deseas probar la infraestructura de contenedores de manera local en tu máquina, clona el repositorio y asegúrate de contar con Docker instalado. Luego, ejecuta en la raíz:

```bash
# Construir las imágenes locales y levantar la solución completa
docker-compose up -d --build

# Verificar el estado de los contenedores y límites de recursos
docker ps
docker stats