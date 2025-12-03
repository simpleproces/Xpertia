# XPERTIA – Asistente de Pertinencia en la Educación Superior

> Solución presentada al **Concurso Datos del Ecosistema 2025** – Portal Datos Abiertos (datos.gov.co)

---

## 1. Resumen ejecutivo

**XPERTIA** es un asistente conversacional que automatiza la elaboración de **estudios de pertinencia de programas académicos** de Instituciones de Educación Superior (IES) en Colombia, usando datos del ecosistema de datos abiertos y modelos de IA generativa.

A partir del **código SNIES** de un programa, el sistema:

1. Consulta y cruza datos abiertos (oferta académica, matrícula, graduados, contexto territorial, etc.).
2. Construye métricas e indicadores de desempeño en **BigQuery**.
3. Genera **tableros interactivos** en Looker Studio.
4. Produce un **análisis narrativo en lenguaje natural** usando un LLM (ChatGPT).
5. Entrega al usuario:
   - Un enlace al informe gráfico (Looker Studio).
   - Un **informe en Word** descargable con el análisis automatizado del programa.

El usuario interactúa únicamente con un **bot conversacional**, que orquesta toda la lógica a través de n8n y servicios en la nube.

---

## 2. Objetivo de la solución

- Facilitar a las IES la elaboración de **estudios de pertinencia** para la creación o renovación de programas académicos.
- Demostrar cómo el **ecosistema de datos abiertos** puede integrarse con herramientas modernas (automatización, BI, IA generativa) para apoyar decisiones de política educativa.
- Reducir tiempos y esfuerzo técnico para equipos académicos que no son expertos en datos, ofreciendo resultados a través de una interfaz conversacional sencilla.

---

## 3. Arquitectura general

La solución sigue una arquitectura modular:

- **Botpress (XPERTIA)** – Asistente conversacional donde el usuario:
  - Ingresa el **código SNIES** del programa target.
  - Recibe el enlace al informe de Looker Studio y al Word generado.

- **n8n** – Motor de orquestación:
  - Recibe las solicitudes de Botpress vía **Webhook**.
  - Ejecuta consultas en **BigQuery**.
  - Llama a la API de **OpenAI (ChatGPT)** para generar el análisis textual.
  - Genera un archivo **Word (.docx)** a partir de una plantilla y lo almacena en Google Drive.
  - Devuelve a Botpress los enlaces generados.

- **BigQuery** – Almacén de datos y capa semántica:
  - Carga y modela los datasets del ecosistema (SNIES, matrícula, instituciones, etc.).
  - Define tablas de hechos y vistas específicas para el estudio de pertinencia, como:
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
  - Publica el documento con acceso mediante enlace; el link se envía al usuario desde el bot.

> *(Opcional)* Aquí puede incluirse un diagrama: `docs/arquitectura.png`.

---

## 4. Flujo de uso (vista del usuario)

1. El usuario abre el asistente conversacional **XPERTIA**.
2. El bot solicita:
   - Código SNIES del programa.
   - (Opcional) otros datos básicos del estudio.
3. Botpress envía una petición HTTP a n8n con:
   ```json
   {
     "estudio_id": "<ID generado por el bot>",
     "cod_snies_programa": "<código SNIES>"
   }
4. n8n:
-Ejecuta consultas en BigQuery para construir las tablas/vistas del estudio.
-Prepara un JSON consolidado con:
  -programa_caracteristicas
  -series_target_detalle
  -series_target_resumen
-Llama a OpenAI (LLM) usando ese JSON.
-Toma el análisis del LLM, lo inyecta en la plantilla Word y sube el .docx a Google Drive.
-Retorna a Botpress:
      {
        "word_url": "<link público al Word>",
        "looker_url": "<link al informe Looker Studio>",
        "analisis_llm": "<texto completo del análisis>",
        "estudio_id": "<id>",
        "cod_snies_programa": "<snies>"
      }
5. El bot responde al usuario con:
   -Enlace al informe visual (Looker Studio).
   -Enlace al informe descargable en Word.
   -(Opcional) un resumen ejecutivo del análisis en el chat.

---

## 5. Datos del ecosistema utilizados

La solución se apoya en datasets oficiales del ecosistema de datos abiertos y en bases consolidadas del SNIES:

MEN_MATRICULA_ESTADISTICA_ES
Portal de Datos Abiertos – estadísticas de matrícula en educación superior.
https://www.datos.gov.co/Educaci-n/MEN_MATRICULA_ESTADISTICA_ES/5wck-szir

MEN_PROGRAMAS_DE_EDUCACIÓN_SUPERIOR
Información de programas académicos: códigos SNIES, niveles, modalidades, áreas de conocimiento, etc.
https://www.datos.gov.co/Educaci-n/MEN_PROGRAMAS_DE_EDUCACI-N_SUPERIOR/upr9-nkiz

MEN_INSTITUCIONES EDUCACIÓN SUPERIOR
Listado de IES con información institucional básica.
https://www.datos.gov.co/Educaci-n/MEN_INSTITUCIONES-EDUCACI-N-SUPERIOR/n5yy-8nav

Métricas de inscritos, admitidos, matriculados y graduados (bases consolidadas SNIES)
Estadísticas históricas del SNIES para inscritos, admitidos, matriculados de primer curso, matriculados totales y graduados.
https://snies.mineducacion.gov.co/portal/ESTADISTICAS/Bases-consolidadas/

Estos datasets se integran en un esquema de datos en BigQuery (staging, dimensiones y hechos) que sirve de base para los estudios de pertinencia automatizados.

---

## 6. Componentes técnicos

6.1 Lenguajes / entornos
 -SQL (BigQuery Standard SQL)
 -n8n (workflows en formato JSON)
 -Botpress Cloud (flows + nodos Execute en JavaScript)
 -Plantillas Word .docx (DocxTemplater)

6.2 Servicios
 -Google BigQuery
 -Google Looker Studio
 -Google Drive (almacenamiento de informes Word)
 -Botpress Cloud
 -n8n
 -OpenAI API (modelos GPT-4.x)

---

## 7. Cómo probar la demo

7.1 Asistente conversacional (Botpress)

🤖 XPERTIA – Bot de Estudio de Pertinencia (demo)
👉 https://cdn.botpress.cloud/webchat/v3.3/shareable.html?configUrl=https://files.bpcontent.cloud/2025/05/20/10/20250520102344-8OV7AZ7I.json

   Para iniciar la conversación el usuario deberá saludar al bot (p.ej. Hola); 
   El bot responderá:
   ¡Hola! Soy tu asistente para Estudios de Pertinencia. ¿Deseas trabajar en un estudio de pertinencia? Por favor responde SI o NO
   El usuario dará click en "SI"
   
   El bot preguntará:
   "¿Deseas iniciar un nuevo estudio o continuar uno existente?" Iniciar nuevo estudio / Continuar estudio un existente
   El usuario dará click en "Iniciar nuevo estudio"
   
   El bot responderá:
   🆔 Tu ID de estudio es: xkbfgu-1764376023742 (p.ej)
   Debe guardarlo si necesita retomar más adelante el estudio
   
   El bot preguntará:
   Seleccione el alcance del estudio:
   El usuario seleccionará la opción: Estudio por un código snies de un programa específico
   
   El bot responderá:
   "Por favor, ingresa el código SNIES del programa que necesites analizar"
   El usuario digitará el codigo SNIES y dara enter
   
   El bot (despues de procesar el flujo en n8n y elaborar informe en looker studio) responderá:
   ✅ Ya generé tu estudio de pertinencia.
   Informe gráfico (tableros): Abrir en Looker Studio (Enlace al informe en Looker Studio.)
   Informe detallado (Word): Descargar informe en Word (Enlace al informe en Word con el análisis generado por IA.)

7.2 Tablero de Looker Studio

📊 Informe demo de desempeño de programa target (codigo SNIES conocido)
   👉 https://lookerstudio.google.com/reporting/964f3987-b8c6-4eaf-aa04-5a24d3e1fb47/page/p_ecj9cnncwd/edit?s=t1a5RXiXPZU
   Nota: El enlace está configurado como informe demo en modo lectura/edición compartida según la configuración de acceso del autor.

📊 Informe demo de desempeño de programa nuevo
   👉 https://lookerstudio.google.com/s/ujdkfBk7M3Q
   Nota: El enlace está configurado como informe demo en modo lectura/edición compartida según la configuración de acceso del autor.

8. Limitaciones y trabajo futuro
      - La versión actual se centra en el análisis del programa target. En fases futuras se proyecta:
      - Incorporar comparación automática con programas similares (competencia) usando mercado y participación.
      - Implementar indicadores avanzados de concentración y diversificación de la oferta.
      - Integrar módulos adicionales para pertinencia laboral a partir de estadísticas del Observatorio Laboral para la Educación (OLE) y otras fuentes.
      - El pipeline está optimizado para un stack específico (BigQuery + Looker Studio + Botpress + n8n), pero la lógica es portable a otros motores SQL y herramientas de visualización.

9. Autoría y contacto
Integrantes:
Carlos Cañas: cahucari@gmail.com
Maricela Botero: maricelabot@gmail.com

