# Estudio de Pertinencia de Programas Académicos usando Datos Abiertos y Asistentes Conversacionales

> Solución presentada al **Concurso Datos del Ecosistema 2025** – Portal Datos Abiertos (datos.gov.co)

---

## 1. Resumen ejecutivo

Esta solución automatiza la elaboración de **estudios de pertinencia de programas académicos** de Instituciones de Educación Superior (IES) en Colombia, utilizando datos del ecosistema de datos abiertos y un asistente conversacional.

A partir del **código SNIES** de un programa, el sistema:

1. Consulta y cruza datos abiertos (oferta académica, matrícula, graduados, población, etc.).
2. Construye métricas e indicadores de desempeño territorial en **BigQuery**.
3. Genera **tableros interactivos** en Looker Studio.
4. Produce un **análisis narrativo en lenguaje natural** (LLM/ChatGPT).
5. Entrega al usuario:
   - Un enlace al informe gráfico (Looker Studio).
   - Un **informe en Word** descargable con el análisis automatizado del programa.

El usuario interactúa únicamente con un **bot conversacional**, que orquesta toda la lógica a través de n8n y servicios en la nube.

---

## 2. Objetivo de la solución

- Facilitar a las IES la elaboración de **estudios de pertinencia** para la creación o renovación de programas académicos.
- Demostrar cómo el **ecosistema de datos abiertos** puede integrarse con herramientas modernas (RPA, BI, LLM) para apoyar decisiones de política educativa.
- Reducir tiempos y esfuerzo técnico para equipos académicos que no son expertos en datos, ofreciendo resultados a través de una interfaz conversacional.

---

## 3. Arquitectura general

La solución sigue una arquitectura modular:

- **Botpress** – Asistente conversacional donde el usuario:
  - Ingresa el **código SNIES** del programa target.
  - Recibe el enlace al informe de Looker Studio y al Word generado.

- **n8n** – Motor de orquestación:
  - Recibe las solicitudes de Botpress vía **Webhook**.
  - Ejecuta consultas en **BigQuery**.
  - Llama a la API de **OpenAI (ChatGPT)** para generar el análisis textual.
  - Genera un archivo **Word (.docx)** a partir de una plantilla y lo almacena en Google Drive.
  - Devuelve a Botpress los enlaces generados.

- **BigQuery** – Almacén de datos y capa semántica:
  - Carga y modela los datasets del ecosistema (SNIES, población, etc.).
  - Define tablas de hechos y vistas específicas para el estudio de pertinencia.
  - Expone vistas como:
    - `estudio_programas_caracteristicas`
    - `estudio_target_series`
    - `mart_share_programa`
    - `mart_share_consolidado`
    - entre otras.

- **Looker Studio** – Visualización:
  - Tablero interactivo para explorar el desempeño del programa target:
    - Inscritos, admitidos, matriculados, graduados.
    - Series históricas por año/semestre.
    - Desagregación territorial (departamento/municipio).

- **OpenAI / ChatGPT** – LLM:
  - Recibe un JSON con características del programa + series históricas.
  - Devuelve un análisis estructurado en secciones:
    1. Contexto del programa e institución.
    2. Comportamiento temporal de las métricas.
    3. Riesgos y alertas.
    4. Recomendaciones para directivos.
    5. Resumen ejecutivo.

- **Google Drive + DocxTemplater**:
  - Usa una plantilla `.docx` para generar el informe en Word con el análisis del LLM.
  - Publica el documento con acceso mediante enlace.

> **Diagrama (sugerido)**  
> Puedes incluir aquí una imagen: `docs/arquitectura.png`

---

## 4. Flujo de uso (vista del usuario)

1. El usuario abre el asistente conversacional (Botpress) en la web.
2. El bot solicita:
   - Código SNIES del programa.
   - (Opcional) otros datos básicos del estudio.
3. Botpress envía un request HTTP a n8n con:
   ```json
   {
     "estudio_id": "<ID generado por el bot>",
     "cod_snies_programa": "<código SNIES>"
   }

