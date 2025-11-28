# 🐾 Animalia: Sistema de Comunicación y Reentrenamiento Multiespecie

**URL del Proyecto:** [http://torete.net/animalia](http://torete.net/animalia)

Este proyecto implementa una arquitectura avanzada para el análisis de comportamiento animal y la comunicación multilingüe, utilizando técnicas de Visión por Computadora, Memoria Persistente (Mem0) y Reentrenamiento Distribuido (RL).

## 🚀 Arquitectura del Sistema

El proyecto se compone de dos microservicios principales que se comunican para crear un ciclo de aprendizaje continuo:

1.  **`microservice_vision`**: Analiza el comportamiento animal y la expresividad, utilizando una jerarquía taxonómica compleja para el reentrenamiento.
2.  **`microservice_video_chat`**: Proporciona un Chat RAG (Retrieval-Augmented Generation) contextualizado en videos, con memoria persistente (Mem0 personalizado) para el usuario.

## ⚙️ Despliegue Local para Usuarios (Uso en el Borde)

Este sistema está diseñado para funcionar en tu dispositivo local (en el **borde**), garantizando que tus datos de interacción se mantengan privados y accesibles. Sigue estos pasos para ponerlo en marcha.

### 1. Requisitos Previos

*   Python 3.10 o superior.
*   `pip` (Gestor de paquetes de Python).
*   Se recomienda usar un entorno virtual para evitar conflictos de dependencias.

### 2. Descarga y Configuración

1.  **Descargar el Proyecto**:
    *   Ve al repositorio de GitHub del proyecto Animalia.
    *   Haz clic en `Code` y luego en `Download ZIP`.

2.  **Descomprimir el Archivo**:
    *   Una vez descargado, descomprime el archivo `Animalia-main.zip` en una carpeta de tu elección.

3.  **Configurar Microservicio de Visión (Animalia Core)**:
    *   Abre una terminal o línea de comandos.
    *   Navega a la carpeta `microservice_vision`:
        ```bash
        cd ruta/a/Animalia-main/microservice_vision
        ```
    *   Instala las dependencias:
        ```bash
        pip install -r requirements.txt
        ```
    *   Inicializa la base de datos con la jerarquía animal:
        ```bash
        python db_init.py
        ```
    *   Inicia el servicio (se ejecutará en el puerto 5001):
        ```bash
        python app.py
        ```

4.  **Configurar Microservicio de Chat (Animalia Chat)**:
    *   Abre **una nueva terminal** (deja la anterior ejecutando el servicio de visión).
    *   Navega a la carpeta `microservice_video_chat`:
        ```bash
        cd ruta/a/Animalia-main/microservice_video_chat
        ```
    *   Instala las dependencias:
        ```bash
        pip install -r requirements.txt
        ```
    *   Inicia el servicio (se ejecutará en el puerto 5002):
        ```bash
        python rag_backend.py
        ```

### 3. Acceso y Uso

*   ¡Listo! Ahora puedes acceder a la aplicación de chat abriendo el archivo `chat_ui.html` que se encuentra en la carpeta `microservice_video_chat` en tu navegador web.

## 💾 Arquitectura de Datos: Local y con Respaldo

*   **Almacenamiento Local (Caché en el Borde)**: Todas tus conversaciones, interacciones y la memoria de la IA se guardan de forma segura en un archivo de base de datos local llamado `user_memory_local.db` dentro de la carpeta `microservice_video_chat`. Esto garantiza que tienes control total sobre tus datos y que el sistema puede funcionar de forma rápida y privada.

*   **Respaldo en Hosting (No Indicado al Usuario)**: Para asegurar la integridad de los datos de entrenamiento y la mejora continua del modelo de IA general, el sistema realiza un respaldo automático y transparente de los datos anonimizados a un servidor central en `torete.net`. Tu información personal no se comparte, solo los patrones de interacción que ayudan a la IA a aprender.

## 🧠 Reentrenamiento y Comunicación Multiespecies

El sistema está diseñado para aprender de tus interacciones. Incluye un vasto conjunto de datos de sonidos de la naturaleza y de la fauna silvestre y doméstica para mejorar la capacidad de la IA de entender y traducir la comunicación animal.

## 📚 Documentación Técnica

Para una configuración avanzada, consulta los manuales detallados dentro de cada microservicio:

*   `microservice_vision/MANUAL_COMUNICACION_ANIMAL_MULTILINGUE.md`
*   `microservice_video_chat/MANUAL_CHAT_RAG.md`
*   `ANALISIS_LIGHTMEM_DOLPHIN_GEMMA.md`

