# 🚀 Airflow Monitoring Agent

## 📋 Descripción

Este proyecto implementa un asistente inteligente para monitorear y gestionar flujos de trabajo (DAGs) en Apache Airflow utilizando modelos de lenguaje avanzados. El agente permite consultar en lenguaje natural el estado de los DAGs, aplicar filtros por diversos criterios y obtener información detallada sobre las ejecuciones programadas.

## ✨ Características principales

- **Consultas en lenguaje natural**: Interactúa con tus DAGs de Airflow usando preguntas en español (o cualquier idioma).
- **Filtrado inteligente**: Busca DAGs por ID, propietario o estado (pausado/activo).
- **Información detallada**: Obtén datos completos sobre los próximos intervalos de ejecución y el historial de cada DAG.
- **Soporte multi-entorno**: Configuración para diferentes entornos (desarrollo, integración, preproducción, producción).
- **Autenticación con Google Cloud**: Integración con servicios de GCP para acceso seguro.
- **Paginación automática**: Gestión eficiente de grandes colecciones de DAGs.

## 🛠️ Tecnologías utilizadas

- **Python**: Lenguaje base de implementación
- **Pydantic-AI**: Framework para la creación de agentes IA
- **Google Gemini 2.0**: Modelo de IA para procesamiento de lenguaje natural
- **Apache Airflow API**: Para acceder a la información de los DAGs
- **HTTPX**: Cliente HTTP asíncrono para Python
- **Asyncio**: Para operaciones asíncronas
- **Colorlog**: Mejora la visualización de logs con colores

## 📦 Requisitos previos

- Python 3.9+
- Acceso a una instancia de Apache Airflow (preferiblemente en Google Cloud Composer)
- Cuenta de Google Cloud con permisos adecuados
- Paquetes Python especificados en `requirements.txt`

## 🚀 Instalación

1. Clona este repositorio:
```bash
git clone https://github.com/tu-usuario/airflow-monitoring-agent.git
cd airflow-monitoring-agent
```

2. Instala las dependencias (usando uv o pip):
```bash
uv install -r requirements.txt
```

3. Configura tus credenciales de GCP si es necesario

## 💻 Uso

El agente se ejecuta desde la línea de comandos, especificando el entorno y si se requiere autenticación:

```bash
uv run agent.py <entorno> <auth>
```

Donde:
- `<entorno>` puede ser: dev, itg, pre, pro
- `<auth>` puede ser: si, no

Ejemplo:
```bash
uv run agent.py dev no
```

Una vez iniciado, el agente te pedirá tu consulta. Algunos ejemplos:

- "Muéstrame todos los DAGs pausados"
- "¿Cuál es el estado del DAG data_processing_dag?"
- "Busca los DAGs que pertenecen al usuario analytics_team"
- "¿Cuándo será la próxima ejecución del DAG etl_workflow?"

## 🏗️ Arquitectura

### Componentes principales

1. **Modelos de datos (Pydantic)**:
   - `DAGStatus`: Modelo para la información de un DAG individual
   - `DAGStatusList`: Contenedor para múltiples estados de DAGs

2. **Agente IA**:
   - Configurado con Gemini 2.0
   - Sistema de prompts diseñado para entender consultas sobre Airflow
   - Herramientas para interactuar con la API de Airflow

3. **Herramientas del agente**:
   - `list_dags`: Recupera todos los DAGs disponibles
   - `get_dag_status`: Obtiene información detallada filtrando por varios criterios

4. **Utilidades**:
   - Autenticación con Google Cloud
   - Manejo de tokens de acceso
   - Configuración de logging mejorado

## 🔍 Detalles de implementación

### Sistema de agente

El agente utiliza un enfoque basado en herramientas que permite al modelo de IA decidir qué acciones tomar en función de la consulta del usuario. El flujo típico es:

1. El usuario hace una consulta en lenguaje natural
2. El modelo analiza la consulta y decide qué información necesita
3. Primero obtiene una lista de todos los DAGs disponibles
4. Determina qué filtros aplicar según la consulta
5. Ejecuta `get_dag_status` con los filtros apropiados
6. Formatea la respuesta para el usuario

### Manejo de paginación

Una característica clave es el manejo eficiente de grandes conjuntos de DAGs mediante paginación:

```python
while True:
    params = {
        'limit': limit,
        'offset': offset
    }
    
    response = await client.get(base_uri, headers=_headers, params=params)
    # Procesamiento...
    
    if len(all_dags) >= total_entries or not current_dags:
        break
        
    offset += limit
```

### Filtrado inteligente

El agente implementa un sistema de filtrado flexible:

```python
if dag_id:
    filtered_dags = [dag for dag in filtered_dags if dag['dag_id'] == dag_id]

if owner:
    filtered_dags = [dag for dag in filtered_dags if owner in dag.get('owners', [])]

if is_paused is not None:
    filtered_dags = [dag for dag in filtered_dags if dag.get('is_paused') == is_paused]
```

## 📊 Ejemplos de salida

Cuando el agente encuentra DAGs que coinciden con tu consulta, presentará información detallada como:

```
Found DAGs:

-------------------
DAGStatus(
    dag_id='data_processing_workflow',
    dag_display_name='Data Processing Pipeline',
    is_paused=False,
    next_dag_run_data_interval_start='2023-11-12T00:00:00+00:00',
    next_dag_run_data_interval_end='2023-11-13T00:00:00+00:00',
    last_dag_run_id='scheduled__2023-11-11T00:00:00+00:00',
    last_dag_run_state='success',
    total_dag_runs=124
)
```

## 🔮 Mejoras futuras

- **Interfaz web**: Desarrollar una interfaz de usuario web para interactuar con el agente
- **Acciones de control**: Permitir acciones como pausar/reanudar DAGs, disparar ejecuciones
- **Alertas proactivas**: Notificaciones sobre fallos o comportamientos anómalos
- **Análisis histórico**: Estadísticas y tendencias de rendimiento
- **Integración con chat**: Conectar con plataformas como Slack o Discord

## 📄 Licencia

Este proyecto está licenciado bajo MIT License - ver el archivo LICENSE para más detalles.

## 👥 Contribuciones

Las contribuciones son bienvenidas. Por favor, abre un issue o pull request para sugerencias, mejoras o correcciones.

---

⭐ Si encuentras útil este proyecto, ¡no dudes en darle una estrella en GitHub! ⭐
