# Avances PT - Edric 
---

## 1. Resumen de cada campo a trabajar

- **VANT** captura imágenes del pavimento en vuelo autónomo (waypoints).
- Un **modelo de IA (CNN con segmentación semántica)** detecta, delimita y clasifica por severidad los baches.
- La **telemetría y las detecciones** (ubicación, severidad, área, marca de tiempo) se transmiten por radio a una estación base y se publican en una **plataforma web geoespacial**.
- Una **aplicación móvil** permite a las cuadrillas de campo consultar y actualizar el estado de atención de cada bache.
- Post-vuelo, las fotografías se procesan de forma local para generar **ortomosaicos georreferenciados**.
- Se propone un **modelo de confiabilidad del enlace inalámbrico** que relaciona la SNR bajo desvanecimiento (Rayleigh/Rician) con la probabilidad de detección del modelo de IA.

## 2. Módulos del sistema (Como esta en el doc)

|Módulo|Nombre|Responsabilidad principal|
|---|---|---|
|M1|Diseño y construcción del VANT|Diseño, ensamble e integración de la plataforma multirrotor y sus subsistemas|
|M2|Sistema de vuelo autónomo|Planificación y ejecución de misiones por waypoints|
|M3|Detección de baches con IA|Detección, segmentación y clasificación por severidad|
|M4|Comunicación y confiabilidad del enlace|Telemetría, envío de detecciones y modelo predictivo de confiabilidad|
|M5|Visualización, gestión y seguimiento|Plataforma web y app móvil sobre API REST + PostGIS|
|M6|Procesamiento fotogramétrico|Ortomosaicos post-vuelo en hardware local|

**Flujos de operación**

1. **Tiempo real:** M2 ejecuta la misión → M3 detecta a bordo → M4 transmite → estación base → nube → mapa web (M5).
2. **Post-vuelo:** extracción manual de fotos → M6 genera el ortomosaico → se incorpora a la plataforma como capa complementaria.

## 3. Repositorios y recursos

| Recurso                         | Descripción                                                                                          | Enlace                                                                                                      |
| ------------------------------- | ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Plataforma web / telemetría** | Frontend, backend (API REST) y base de datos geoespacial                                             | [telemetry-platform](https://github.com/Edriczzzz/telemetry-platform)                                       |
| **Aplicación móvil**            | App para cuadrillas de mantenimiento vial                                                            | [pt-movil](https://github.com/Edriczzzz/pt-movil)                                                           |
| **Modelo de enlace FPV**        | Código del trabajo de Rodo base de cálculo para el modelo matemático de confiabilidad que proponemos | [Modelo_Enlace-FPV](https://github.com/Edriczzzz/Modelo_Enlace-FPV)                                         |
| **Dataset (Google Drive)**      | Dataset que construí (hasta el momento) para entrenamiento del modelo de IA (Perdón por usar tu drive chompy)                         | [Carpeta de Drive](https://drive.google.com/drive/folders/1VtqtsZU4hTLdu7hOV5QhSXCYX-xNzavu?usp=drive_link) |
| **Notebook de IA (Colab)**      | Entrenamiento y corrida del modelo de detección                                                      | [Google Colab](https://colab.research.google.com/drive/1jrzlB8fLN-ea9TWcWz8I6WnFu4-eDLPH?usp=sharing)       |

> Nota: el repositorio del modelo de enlace corresponde a el trabajo que le hice de la beifi a rodo; lo usaremos como referencia de cálculo, y nuestra propuesta debería se una aproximación derivada y evaluada con el mismo ejemplo de ese paper.

## 4. Plataforma web

**Stack**

- **Frontend:** React + TypeScript + Vite, Leaflet/React-Leaflet para mapas, Tailwind CSS con shadcn/ui, React Router, TanStack Query, Zod + React Hook Form.
- **Backend:** Python con FastAPI, Pydantic, SQLAlchemy 2 y Alembic (migraciones).
- **Base de datos:** PostgreSQL con extensión **PostGIS** para coordenadas, trayectorias y áreas de detección.
- **API:** REST bajo `/api/v1`, con WebSockets para actualizaciones en vivo.
- **Entorno local:** Docker Compose (desarrollo en Windows con Docker Desktop).

**Roles previstos:** Administrador, Operador, Personal de campo y Usuario de consulta (autorización aplicada en el backend).

**Decisiones de alcance**

- La primera entrega **no incluye comandos de vuelo** (armado, despegue, aterrizaje). Aunque es importante ir hgaciendo pruebas de como hacerlo
- Las fotos y el procesamiento fotogramétrico son manuales/locales.
- Los datos de demostración siempre se identifican como tales.
- Despliegue económico para proyecto debera de ser en algo como Render, Railway o equivalente.

**Avances concluidos**

- Fase 1 (análisis y diseño): .se puede mejorar pero esta ok
- Fase 2: infraestructura y servidor corriendo en entorno local.
- Fase 3 (primera parte): panel, mapa, inventario y detalle de bache funcionando; el diseño visual se mejora más adelante.

## 5. Aplicación móvil — `pt-movil`
Esto es una primer versión, esta sujeta a pruebas pero dado el entorno del dev web me parece es una muy buena solución para el consumo de las mismas apis que se configuren, pero esta sujeta a cambios

**Stack:** React Native + Expo + TypeScript, Expo Router, React Native Paper, React Native Maps, TanStack Query, cliente HTTP centralizado, React Hook Form + Zod, Expo SecureStore, ESLint/Prettier. Pruebas en teléfono Android físico con Expo Go.

**Funciones:** inicio, mapa, inventario, detalle de bache, registro de atención, trabajos e historial. Interfaz en español con navegación inferior.

**Modelo de datos (conceptual)**

- **Severidad:** baja / media / alta.
- **Estado de atención:** pendiente / en revisión / atendido / descartado (provisional hasta alinearlo con el backend).

**Arquitectura:** capa de servicios intercambiable **mock / API real**, conectada a la misma API FastAPI/PostGIS que la web. Los endpoints `/api/v1/detections` y `/api/v1/work-orders` son propuestas de diseño,

## 6. Modelo de IA — Dataset y Colab

- Detección y **segmentación semántica** de baches con una CNN, con clasificación de severidad.
- Dataset consolidado a partir de conjuntos públicos de detección aérea de deterioros viales, complementado con imágenes capturadas localmente en vialidades de la CDMX; etiquetado con polígonos y aumento de datos (iluminación, altitud, ángulo).
- **Meta de desempeño:** mAP ≥ **80 %**  (actualmente consegui 63.6%)sobre el conjunto de prueba en inferencia local (sin degradación del canal), que sirve de valor de referencia para la prueba de confiabilidad del enlace.
- **Entrenamiento y ejecución:** notebook en Google Colab.
- **Resultados actuales:** 

## 7. Modelo de confiabilidad del enlace

Cadena propuesta para pt: **condiciones del canal → confiabilidad del enlace → calidad de los datos → confiabilidad de la detección**.

Métrica unificada:

```
R_sys(d) = R_comm(d) × R_det(d)
```

Pérdida de trayectoria logarítmica:

```
PL(d) = PL(d0) + 10·n·log10(d/d0)
```

Confiabilidad del enlace (probabilidad de que la SNR supere el umbral γ_th):

```
R_comm(d) = P(γ(t) > γ_th)
```

Para desvanecimiento Rayleigh:

```
R_comm(d) = exp(−γ_th / γ̄(d))
```

**Validación experimental prevista:** VANT en vuelo estacionario a 10, 20, 30, 50, 75 y 100 m de la estación base (enlace de 915 MHz), registrando RSSI, SNR estimada y tasa de detección del modelo de IA, para contrastar lo predicho con lo medido.
## 8. Avances documentados

1. **Plataforma web:** clonar [telemetry-platform](https://github.com/Edriczzzz/telemetry-platform) y levantar el entorno local con Docker Compose (ver el README del repositorio).
2. **App móvil:** clonar [pt-movil](https://github.com/Edriczzzz/pt-movil), instalar dependencias y abrir con Expo Go en un teléfono (android)
3. **Modelo de IA:** abrir el [notebook de Colab](https://colab.research.google.com/drive/1jrzlB8fLN-ea9TWcWz8I6WnFu4-eDLPH?usp=sharing) y montar el [dataset en Drive](https://drive.google.com/drive/folders/1VtqtsZU4hTLdu7hOV5QhSXCYX-xNzavu?usp=drive_link).
4. **Modelo de enlace:** revisar [Modelo_Enlace-FPV](https://github.com/Edriczzzz/Modelo_Enlace-FPV) como base de cálculo de la confiabilidad.
