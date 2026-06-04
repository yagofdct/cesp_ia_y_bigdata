### 1. Arquitectura del Sistema
#### Ingesta de datos
Se acciona un trigger manual. Este envía los documentos al cargador de datos, donde son segmentados mediante un algoritmo de chunking. Cada fragmento se transforma en un embedding utilizando OpenAI y posteriormente se almacena en Pinecone junto con su contenido textual para permitir búsquedas semánticas posteriores.

Trigger manual → Documentos → Chunking → Embeddings (OpenAI text-embedding-3-small) → Pinecone Vector Database

#### Asistente RAG
El usuario envía un mensaje mediante Telegram. El AI Agent recibe la consulta y utiliza la herramienta de búsqueda semántica conectada a Pinecone. Para ello, la consulta se transforma en un embedding mediante OpenAI y se compara con los embeddings almacenados en la base de datos vectorial. Pinecone devuelve los fragmentos más relevantes, que son incorporados al contexto del modelo. Finalmente, el modelo genera una respuesta utilizando tanto la pregunta original como la documentación recuperada, y esta respuesta es enviada de vuelta al usuario a través de Telegram.

Telegram Trigger → AI Agent → Embedding de la consulta → Búsqueda semántica en Pinecone → Recuperación de contexto → Generación de respuesta (nvidia/nemotron3-ultra) → Telegram Send Message


### 2. Decisiones tomadas
- He preferido usar n8n en lugar de LangChain. Esto se debe a que n8n es mucho más rápido y cómodo de configurar, además de que para este caso en particular LangChain no presenta ninguna ventaja frente a n8n.
- El tamaño de los chunks que he configurado es de 300 caracteres. Este valor resulta adecuado para este tipo de documentación, ya que los apartados contienen más información que una simple frase, pero no son lo suficientemente extensos como para requerir fragmentos de mayor tamaño.
- En cada consulta se recuperan un máximo de cuatro fragmentos de documentación (k = 4). Esta configuración permite proporcionar al modelo el contexto suficiente como para responder preguntas que requieran de información distribuida entre varios fragmentos, evitando al mismo tiempo introducir una cantidad excesiva de información que pueda afectar negativamente a la precisión de las respuestas.
- El modelo que seleccioné fue nvidia/nemotron3-ultra. Lo escogí porque ofrece acceso gratuito y un rendimiento adecuado para las necesidades de este proyecto. Su capacidad es suficiente para interpretar las consultas del usuario, utilizar el contexto recuperado desde la base de datos vectorial y generar respuestas coherentes y relevantes.


### 3. Ejemplos de funcionamiento
- En la carpeta "capturas" he añadido imágenes tanto del workflow y la configuración de los nodos en n8n, como de la base de datos de Pinecone.
- También se han documentado 5 pruebas en la carpeta "pruebas", con capturas de la conversación de telegram, los fragmentos de documentación extraidos y una evaluación de los resultados obtenidos.

### 4. Mejoras propuestas
Como posibles mejoras del sistema RAG desarrollado, se podrían implementar las siguientes:
- Configurar un proceso de ingesta automática de documentos desde sistemas de almacenamiento en la nube, como Google Drive o Microsoft SharePoint, evitando así la carga manual de la documentación.
- Añadir soporte multilingüe, permitiendo que el sistema procese documentos y responda consultas en distintos idiomas.
- Incorporar un mecanismo de re-ranking de resultados, con el objetivo de reordenar los fragmentos recuperados por Pinecone y seleccionar aquellos que sean realmente más relevantes para la consulta del usuario.
- Ampliar el soporte a diferentes tipos de documentos, como PDF, DOCX, XLSX o CSV, permitiendo trabajar con fuentes de información más variadas.
- Sustituir el modelo de lenguaje utilizado por uno más potente, mejorando la calidad, precisión y coherencia de las respuestas generadas.