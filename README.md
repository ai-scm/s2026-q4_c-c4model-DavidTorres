<div align="center">

# 📊 DailyPulse — Arquitectura de Software (Modelo C4)

**Sistema automatizado para el control, validación y seguimiento de Dailies Scrum integrado con Microsoft Teams para Acme Corp.**

![Arquitectura](https://img.shields.io/badge/Arquitectura-C4_Model-0078D4?style=for-the-badge)
![Integración](https://img.shields.io/badge/Integración-Microsoft_Teams-6264A7?style=for-the-badge)
![Enfoque](https://img.shields.io/badge/Enfoque-Agnóstico_a_Tecnología-2EA44F?style=for-the-badge)

</div>

---

## 📑 Tabla de Contenidos
1. [Introducción al Modelo C4](#-introducción-al-modelo-c4)
2. [Justificación y Lógica de Estructuración](#-justificación-y-lógica-de-estructuración)
3. [Nivel 1: Contexto del Sistema (C1)](#-nivel-1-contexto-del-sistema-c1)
4. [Nivel 2: Contenedores (C2)](#-nivel-2-contenedores-c2)
5. [Nivel 3: Componentes (C3)](#-nivel-3-componentes-c3)
6. [Entregables del Repositorio](#-entregables-del-repositorio)

---

## 🧭 Introducción al Modelo C4

El **Modelo C4** (*Context, Containers, Components & Code*) es una técnica de visualización jerárquica de arquitectura de software que permite comprender un sistema desde distintas perspectivas, yendo de lo más general a lo más específico. Su principal fortaleza radica en facilitar la comunicación entre perfiles de negocio, gestión y desarrollo técnico.

En este proyecto se abordan los tres primeros niveles de abstracción:

| Nivel | Nombre | Propósito en la Arquitectura |
| :--- | :--- | :--- |
| **C1** | **Contexto del Sistema** | Muestra el panorama general ("el bosque"): presenta el sistema como una unidad central rodeada por los usuarios que interactúan con él y los sistemas externos de los que depende. |
| **C2** | **Contenedores** | Abre la caja del sistema para mostrar las unidades ejecutables o de almacenamiento de datos (interfaces web embebidas, servicios backend, tareas programadas, bases de datos) y cómo se comunican. |
| **C3** | **Componentes** | Hace *zoom* dentro de un contenedor específico (el backend principal) para detallar sus módulos lógicos internos, sus responsabilidades de negocio y el flujo de datos entre ellos. |

---

## 🧠 Justificación y Lógica de Estructuración

La arquitectura de **DailyPulse** se diseñó bajo los principios de **simplicidad operacional, baja fricción para el usuario y separación clara de responsabilidades**, evitando la sobreingeniería para facilitar su implementación en futuros sprints. Dado que Acme Corp opera sobre Microsoft Teams, se decidió integrar toda la experiencia de usuario dentro de dicha plataforma combinando un **Bot conversacional** (para notificaciones *push* y formularios diarios con casillas predefinidas, inspirado en la dinámica asíncrona de *Standup & Prosper*) y una **Interfaz Web Embebida** (pestaña dentro de Teams para la configuración de equipos, horarios y visualización de métricas). De este modo, delegamos la autenticación, gestión de identidad y mensajería a MS Teams, reduciendo nuestro desarrollo propio a un núcleo ligero compuesto por una interfaz embebida, un planificador de tareas programadas, un backend centralizado y una base de datos.

A nivel interno del backend (**Motor de Dailies**), la arquitectura se organizó estrictamente alrededor de las reglas de negocio solicitadas por Acme Corp. Se separaron los puntos de entrada (controladores) de la lógica de dominio y las reglas de validación. Al aislar la **Detección de Estancamientos** (análisis de repetición temática en tres reportes consecutivos) y la **Auditoría de Cumplimiento** (verificación de reportes omitidos) en componentes independientes, el sistema permite ajustar los criterios de detección de bloqueos o los tiempos de tolerancia sin impactar la recepción de formularios, la publicación en canales ni la capa de persistencia.

> [!NOTE]
> **Decisión de Diseño (Agnóstico a Tecnología):** Siguiendo los lineamientos del Sprint Review, los diagramas se mantienen libres de tecnologías o proveedores específicos (ej. sin especificar motores de BD o frameworks) para centrar la alineación en los flujos de datos y reglas de negocio.

---

## 🌐 Nivel 1: Contexto del Sistema (`C1`)

Define el alcance de **DailyPulse** y su interacción con los usuarios de Acme Corp, utilizando **Microsoft Teams** como único punto de contacto frontal.

### Actores y Sistemas
* 👤 **Integrante del Equipo Scrum `[Persona]`:** Miembro del equipo que recibe recordatorios diarios del bot y diligencia su reporte de estado en casillas predefinidas.
* 👤 **Scrum Master / Líder `[Persona]`:** Configura los horarios y canales del equipo mediante la pestaña embebida, supervisa el cumplimiento diario y recibe alertas tempranas de bloqueo.
* 🏢 **Microsoft Teams `[Sistema de Software Externo]`:** Plataforma de mensajería corporativa usada como interfaz de interacción (chat directo, canales grupales y pestañas embebidas) y proveedor de identidad.
* 🔷 **DailyPulse `[Sistema de Software]`:** Sistema automatizado que recolecta, valida, analiza y publica los reportes diarios Scrum.

### Flujo de Interacciones
1. **Integrante del Equipo Scrum** $\rightarrow$ **MS Teams:** Recibe notificaciones del bot y diligencia su reporte diario.
2. **Scrum Master / Líder** $\rightarrow$ **MS Teams:** Configura horarios en pestaña embebida, lee reportes en el canal y recibe alertas.
3. **MS Teams** $\rightarrow$ **DailyPulse:** Envía respuestas de formularios, configuraciones e identidad del usuario.
4. **DailyPulse** $\rightarrow$ **MS Teams:** Envía recordatorios, publica resúmenes en canales y emite alertas `[API/Bot]`.

---

## 📦 Nivel 2: Contenedores (`C2`)

Al abrir el límite del sistema **DailyPulse**, se identifican cuatro contenedores encargados de soportar la operación diaria:

| Contenedor | Tipo | Responsabilidad Principal |
| :--- | :--- | :--- |
| **Portal / App Embebida Teams** | `Aplicación Web` | Provee la interfaz visual embebida dentro de una pestaña de Teams para administrar equipos, horarios y métricas, además de definir las tarjetas interactivas del formulario diario. |
| **Motor de Dailies (API Core)** | `API / Backend` | Núcleo central del sistema. Expone endpoints para el bot y la interfaz embebida, ejecuta la lógica de validación de reportes, detección de bloqueos y auditoría de faltantes. |
| **Planificador de Recordatorios (Scheduler)** | `Tarea Programada` | Ejecuta tareas programadas según la zona horaria de cada equipo para disparar los recordatorios diarios y la revisión de incumplimientos. |
| **Base de Datos de Dailies** | `Base de Datos` | Almacena de forma persistente equipos, miembros, horarios configurados, historial de reportes diarios, estado de cumplimiento y alertas generadas. |

### Resumen de Comunicación entre Contenedores
* **MS Teams** carga el **Portal / App Embebida Teams** en una pestaña (que consulta/guarda parámetros vía API en el **Motor de Dailies**) y envía las respuestas de las tarjetas interactivas hacia el **Motor de Dailies** vía Webhook/Bot.
* **Planificador de Recordatorios** dispara hacia el **Motor de Dailies** los eventos de *"Iniciar Daily"* y *"Auditar Faltantes"* según el horario configurado.
* **Motor de Dailies** lee y escribe registros en la **Base de Datos de Dailies**, y envía notificaciones *push*, resúmenes en canales y alertas hacia **MS Teams**.

---

## ⚙️ Nivel 3: Componentes (`C3`)

Acercamiento interno al contenedor **Motor de Dailies (API Core)**, desglosado en **8 componentes** organizados por responsabilidad:

### 1. Capa de Entrada (Controladores)
* **`Controlador de Bot y Webhooks`** `[Controlador API]`: Recibe los envíos de formularios (*dailies*) y eventos de identidad provenientes de MS Teams, validando la autenticidad de cada petición.
* **`Controlador de Configuración`** `[Controlador API]`: Expone los endpoints para que la interfaz embebida cree o edite equipos, miembros, zonas horarias y horas de envío.

### 2. Capa de Lógica de Negocio y Reglas de Dominio
* **`Gestor de Equipos y Horarios`** `[Servicio de Dominio]`: Administra las reglas de membresía de los equipos Scrum, horarios programados y canales destino de publicación.
* **`Orquestador de Reportes Diarios`** `[Servicio de Dominio]`: Estructura las casillas del reporte recibido (*ayer*, *hoy*, *bloqueos*), coordina su persistencia y dispara tanto la publicación como el análisis de estancamiento.
* **`Detector de Estancamientos`** `[Motor de Reglas]`: Evalúa los últimos **3 reportes consecutivos** del integrante; detecta repetición de temática o bloqueo y dispara una alerta temprana al Scrum Master.
* **`Auditor de Cumplimiento`** `[Motor de Reglas]`: Se ejecuta al cumplirse el plazo diario; cruza los miembros activos del equipo contra los reportes recibidos para detectar omisiones.

### 3. Capa de Integración y Persistencia
* **`Adaptador de Mensajería Teams`** `[Cliente de Integración]`: Formatea tarjetas adaptativas y envía mensajes directos, resúmenes en canales y alertas de bloqueo u omisión hacia la API de MS Teams.
* **`Repositorio de Datos`** `[Acceso a Datos]`: Abstrae las consultas y escrituras de equipos, reportes históricos, estado de cumplimiento y alertas en la **Base de Datos de Dailies**.

---

## 📁 Entregables del Repositorio

* `dailypulse-c4.drawio` — Archivo editable con los tres niveles diagramados en pestañas separadas (`C1 - Contexto`, `C2 - Contenedores`, `C3 - Componentes`).
* `README.md` — Documentación descriptiva y justificación arquitectónica de la solución.
