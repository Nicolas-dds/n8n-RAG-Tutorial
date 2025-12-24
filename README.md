# n8n-RAG-Tutorial

Esta guía proporciona instrucciones sobre cómo construir un potente agente de IA usando n8n y PostgreSQL. Esta configuración permite el acceso a documentos, historial de conversaciones y recuperación de información web en tiempo real utilizando los créditos gratuitos de OpenAI que n8n proporciona al principio.

## 1. Configuración Inicial

No necesitas instalar modelos locales para esta configuración. n8n viene con créditos gratuitos de OpenAI que puedes usar para comenzar.

## 2. PostgreSQL Setup

PostgreSQL servirá como tu base de datos de búsqueda vectorial y almacenará el historial de conversaciones y el contenido de documentos. Usaremos Docker para ejecutar PostgreSQL localmente.

- Ve a docker.com y descarga e instala Docker Desktop para tu sistema operativo.
- Después de iniciar Docker Desktop, abre tu símbolo del sistema o terminal.
- Ejecuta el siguiente comando de Docker para configurar tu contenedor de PostgreSQL. Este comando crea una base de datos PostgreSQL con una extensión vectorial aplicada automáticamente.
```bash
docker run --name pgvector-container -e POSTGRES_USER=miusuario -e POSTGRES_PASSWORD=micontraseña -e POSTGRES_DB=mibasededatos -p 5432:5432 -d ankane/pgvector
```

- Puedes cambiar `miusuario`, `micontraseña` y `mibasededatos` a tus valores preferidos.
- Verifica que tu base de datos esté ejecutándose en Docker Desktop.

## 3. Configuración de n8n

n8n es una herramienta de automatización de flujos de trabajo que orquestará tu agente de IA.

- Instala NodeJS yendo a nodejs.org y descargando e instalando Node para tu sistema operativo.
- Abre tu símbolo del sistema y ejecuta el siguiente comando para descargar e iniciar n8n:
```bash
npx n8n start
```

- Accede a n8n copiando la URL proporcionada en tu terminal (por ejemplo, http://localhost:5678).

## 4. Crear Flujo de Trabajo del Agente de IA

Una vez en n8n, crearás un flujo de trabajo para tu agente de IA.

- Haz clic en "Create Workflow" (Crear Flujo de Trabajo) y dale un nombre (por ejemplo, mi agente de IA).
- **Agregar un Nodo Disparador**:
  - Haz clic en el botón + y busca **On Chat Message** (En Mensaje de Chat). Agrega este nodo.
- **Agregar un Nodo de Agente de IA**:
  - Haz clic en el botón + junto al nodo On Chat Message y selecciona **Advanced AI > AI Agent** (IA Avanzada > Agente de IA).
  - En la configuración del Agente de IA, haz clic en "Add Option" (Agregar Opción) y agrega un **System Message** (Mensaje del Sistema).
  - Establece el mensaje del sistema: "Eres un asistente útil. Responde de manera natural y amigable."

## 5. Asignar LLM usando Créditos Gratuitos de n8n

- Desde el nodo AI Agent, haz clic en el botón + junto a "Chat Model" (Modelo de Chat).
- Busca y agrega el nodo **OpenAI Chat Model** (Modelo de Chat OpenAI).
- **Configurar Credenciales**:
  - Haz clic en "Create New Credential" (Crear Nueva Credencial).
  - n8n te ofrecerá usar los **créditos gratuitos de OpenAI** que proporcionan al inicio. Selecciona esta opción.
  - Si no aparece automáticamente, busca la opción "Use n8n's AI credits" o "Usar créditos de IA de n8n".
  - Guarda la credencial.
- Selecciona el modelo que desees (por ejemplo, **gpt-4o-mini** o **gpt-3.5-turbo** para ahorrar créditos) del menú desplegable "Model" (Modelo).
- Renombra el nodo a **OpenAI** para mayor claridad.

## 6. Configurar Memoria de Chat

Para mantener el historial de conversaciones, usaremos PostgreSQL.

- Desde el nodo AI Agent, haz clic en el botón + junto a "Memory" (Memoria).
- Busca y agrega el nodo **Postgres Chat Memory** (Memoria de Chat Postgres).
- **Configurar Credenciales**:
  - Haz clic en "Create New Credential" (Crear Nueva Credencial).
  - Establece el Host como `localhost` (o `127.0.0.1`).
  - Ingresa tu Nombre de Base de Datos, Usuario y Contraseña (por ejemplo, `mibasededatos`, `miusuario`, `micontraseña`) que configuraste en Docker.
  - Asegúrate de que el Puerto sea `5432`.
  - Guarda la credencial.
- Renombra el nodo a **Memory** (Memoria) para mayor claridad.

## 7. Agregar Herramientas

Las herramientas permiten que tu agente interactúe con sistemas externos o acceda a una base de conocimiento personalizada.

### Agregar Base de Conocimiento (RAG):

- Desde el nodo AI Agent, haz clic en el botón + junto a "Tools" (Herramientas).
- Bajo "Vector Stores" (Almacenes Vectoriales), agrega el nodo **Postgres PG Vector Store** (Almacén Vectorial PG de Postgres).
- Selecciona las credenciales de PostgreSQL que creaste anteriormente.
- Para el Modo de Operación, selecciona **Retrieve Documents** (Recuperar Documentos).
- Establece el Nombre como `knowledge base` (base de conocimiento).
- En la Descripción, indica al agente cuándo usar esta herramienta (por ejemplo, "Usa esta herramienta cuando te hagan preguntas de la base de conocimiento personalizada.").
- Establece el Nombre de Tabla (por ejemplo, `n8n_tutorial`). Este nombre debe coincidir con la tabla donde se almacenarán tus documentos.
- Establece el Límite en `4` (o tu número preferido de documentos a devolver).

### Agregar Modelo de Incrustación usando Créditos de n8n:

- Conecta el nodo Postgres PG Vector Store a un nodo **Embeddings OpenAI** (Incrustaciones OpenAI).
- Usa las mismas credenciales de los créditos gratuitos de n8n.
- Para el Modelo de Incrustación, selecciona `text-embedding-ada-002` o `text-embedding-3-small`.

### Agregar Herramienta de Búsqueda Web (Opcional):

- Desde el nodo AI Agent, haz clic en el botón + junto a "Tools" (Herramientas).
- Busca y agrega el nodo **SER API**.
- **Configurar Credenciales**:
  - Haz clic en "Create New Credential" (Crear Nueva Credencial).
  - Ve a serpapi.com para registrar una cuenta y obtener una clave API.
  - Agrega tu Clave API a n8n y guarda.
- Renombra tus nodos de herramientas (por ejemplo, Base de Conocimiento, Búsqueda Web).
- (Opcional) Agrega una "Sticky Note" (Nota Adhesiva) (encontrada bajo "Utilities" o Utilidades) para organizar tu flujo de trabajo visualmente.

## 8. Agregar Documentos a la Base de Datos Vectorial

Para hacer útil tu base de conocimiento personalizada, necesitas cargar documentos en ella.

- Regresa a tu panel de n8n y crea un nuevo flujo de trabajo.
- Llama a este flujo de trabajo `add documents` (agregar documentos).
- **Agregar un Nodo Disparador de Envío de Formulario**:
  - Dale al formulario un Título (por ejemplo, Agregar Documentos).
  - Agrega un campo llamado **File** (Archivo), establece su Tipo como **File** (Archivo), deshabilita Archivos Múltiples y habilita Campo Requerido.
- **Agregar un Nodo Postgres PG Vector Store**:
  - Conéctalo al nodo On Form Submission.
  - Selecciona tus credenciales de PostgreSQL.
  - Para Acción, selecciona **Add Documents to Vector Store** (Agregar Documentos al Almacén Vectorial).
  - Para Modo de Operación, selecciona **Insert Documents** (Insertar Documentos).
  - Establece el Nombre de Tabla con el mismo nombre exacto que usaste en la Base de Conocimiento de tu Agente de IA (por ejemplo, `n8n_tutorial`).
- **Agregar un Nodo Embeddings OpenAI**:
  - Conéctalo al nodo Postgres PG Vector Store.
  - Usa las credenciales de los créditos gratuitos de n8n.
  - Selecciona `text-embedding-ada-002` o `text-embedding-3-small` para el Modelo de Incrustación.
  - Para Documento, selecciona **Default Data Loader** (Cargador de Datos Predeterminado).
  - Cambia el Tipo de JSON a **Binary** (Binario).
- **Agregar un Nodo Recursive Character Text Splitter** (Divisor de Texto de Caracteres Recursivo):
  - Conéctalo al nodo Embeddings OpenAI. Puedes dejar los valores predeterminados.
- **Probar el flujo de trabajo**:
  - Haz clic en "Test Workflow" (Probar Flujo de Trabajo) en el flujo de trabajo add documents.
  - Selecciona un archivo (por ejemplo, PDF, CSV, documento de Word) y envíalo. Esto cargará tus datos en la base de datos.

## Nota sobre los Créditos Gratuitos

n8n proporciona créditos gratuitos de OpenAI al inicio para que puedas comenzar a experimentar sin costo. Una vez que uses estos créditos, necesitarás:
- Agregar tu propia API key de OpenAI, o
- Cambiar a usar modelos locales con Ollama (como en la guía original)

¡Ahora tu agente de IA está listo para usar! Abre la interfaz de chat en tu flujo de trabajo mi agente de IA para comenzar a interactuar con él, hacer preguntas desde tu base de conocimiento y realizar búsquedas web.
