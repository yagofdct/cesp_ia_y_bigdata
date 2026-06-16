# Documento explicativo - Servidor MCP para Gmail

## 1. Descripcion general

En esta practica se ha desarrollado un servidor MCP llamado **Gmail MCP Server** que permite integrar Gmail con Claude Desktop mediante el Model Context Protocol. El servidor esta implementado en Python usando `FastMCP` y expone distintas capacidades de Gmail para que el modelo pueda consultar correos recientes, enviar mensajes, acceder al perfil de la cuenta autenticada y utilizar una plantilla de redaccion de emails.

El proyecto se compone principalmente del archivo `gmail_mcp_server.py`, donde se define el servidor MCP, la autenticacion OAuth con Google y los distintos componentes MCP. Tambien se incluyen `requirements.txt` con las dependencias necesarias y los archivos locales de autenticacion `credentials.json` y `token.json`, que no deben subirse a un repositorio publico porque contienen informacion sensible.

## 2. Flujo OAuth implementado

La autenticacion se realiza mediante OAuth 2.0, utilizando credenciales creadas en Google Cloud Console para una aplicacion de escritorio. Para poder acceder a Gmail se habilito la Gmail API y se configuraron los permisos necesarios en la pantalla de consentimiento OAuth.

El servidor solicita los siguientes scopes:

- `https://www.googleapis.com/auth/gmail.readonly`, para leer correos de la bandeja de entrada.
- `https://www.googleapis.com/auth/gmail.send`, para enviar correos desde la cuenta autenticada.
- `https://www.googleapis.com/auth/gmail.modify`, para permitir operaciones de modificacion sobre Gmail (Actualmente está solicitado, pero el servidor actual no modifica correos.)

La funcion `get_gmail_service()` centraliza todo el proceso de autenticacion. En primer lugar calcula las rutas de `credentials.json` y `token.json` a partir de la ubicacion real del archivo Python, usando `BASE_DIR`. Esto evita problemas cuando el servidor se ejecuta desde Claude Desktop o desde otra carpeta distinta al directorio del proyecto.

En la primera ejecucion, el flujo seguido es el siguiente:

1. Se inicia el flujo OAuth local.
2. Se abre una pestaña de navegador para autorizar el acceso de la aplicación a los recursos de Gmail.
3. Tras autorizar la aplicacion en el navegador, el token se guarda en un archivo `token.json`, el cual se usa en futuras ejecuciones.
4. Finalmente se crea y devuelve el cliente de Gmail con `build("gmail", "v1", credentials=creds)`.

En ejecuciones posteriores, antes de iniciar de nuevo el flujo OAuth, la funcion comprueba si ya existe `token.json`. Si el token sigue siendo valido, lo reutiliza directamente y si el token ha caducado pero dispone de refresh_token, se renueva automáticamente antes de volver a pedir autorización. Gracias a esto, la autorizacion manual solo es necesaria la primera vez o cuando el token no puede reutilizarse.

## 3. Componentes del servidor MCP

El servidor implementa las tres primitivas principales vistas en la unidad: tools, resources y prompts.

### Tools

Las tools son acciones que Claude puede ejecutar a traves del servidor MCP.

**`list_emails(max_results: int = 10)`**

Esta herramienta lista los correos mas recientes de la bandeja de entrada. Usa el servicio de Gmail autenticado y llama a `users().messages().list(...)` con `labelIds=["INBOX"]`. Despues obtiene los metadatos de cada mensaje con `users().messages().get(...)`, recuperando las cabeceras `From`, `Subject` y `Date`.

El resultado se devuelve como texto, incluyendo el ID del mensaje, remitente, asunto y fecha. Si no se encuentran mensajes, la herramienta devuelve un mensaje indicando que no hay emails disponibles.

**`send_email(to: str, subject: str, body: str)`**

Esta herramienta permite enviar un correo desde la cuenta autenticada. Construye el mensaje con `MIMEText`, asigna el destinatario y el asunto, y despues codifica el contenido en base64 URL-safe, que es el formato requerido por la Gmail API. Finalmente llama a `users().messages().send(...)` y devuelve una confirmacion con el ID del mensaje enviado.

### Resource

Los resources exponen informacion consultable por el cliente MCP.

**`gmail://profile`**

El recurso `get_profile()` consulta el perfil de la cuenta autenticada mediante `users().getProfile(userId="me")`. Devuelve la direccion de email, el numero total de mensajes y el numero total de hilos. Esto permite a Claude acceder a informacion general de la cuenta sin ejecutar una accion compleja como enviar un correo.

### Prompt

Los prompts proporcionan plantillas reutilizables para guiar al modelo.

**`redactar_email(destinatario: str, tema: str, tono: str = "profesional")`**

Este prompt genera una instruccion estructurada para redactar un email. Recibe el destinatario, el tema y el tono deseado, y pide al modelo que produzca un mensaje claro, conciso y adecuado al contexto, incluyendo saludo, cuerpo y despedida. Esta plantilla facilita que Claude redacte correos de forma consistente antes de enviarlos con la herramienta `send_email`, si el usuario decide hacerlo.

## 4. Dificultades encontradas durante el desarrollo

Durante el desarrollo del proyecto me he encontrado con varias dificultades relacionadas principalmente con la configuración de OAuth, la integración con Gmail y la puesta en marcha del servidor MCP en Claude Desktop.

La primera dificultad apareció durante la configuración de la pantalla de consentimiento OAuth en Google Cloud. Las instrucciones que se me proporcionaron hacían referencia a una interfaz antigua, en la que aparecían apartados y opciones las cuales ya no existen. Esto hizo necesario adaptar los pasos originales y localizar en la documentación oficial de Google las opciones equivalentes en la nueva interfaz. Apartados y secciones como "APIs y servicios" > "Pantalla de consentimiento OAuth", "Permisos" y "Usuarios de prueba" han sufrido cambios significativos tanto de nombre como de ubicación en la interfaz.

Una vez Claude Desktop consiguió cargar la integración, apareció un nuevo error indicando que no se encontraba el archivo credentials.json. Este problema se debía a que el script estaba buscando el archivo mediante una ruta relativa, por lo que Python intentaba localizarlo desde el directorio de ejecución de Claude Desktop y no desde la carpeta real del proyecto. Para solucionarlo, se añadio `BASE_DIR` para localizar `credentials.json` y `token.json` de forma robusta usando la ruta absoluta del archivo `gmail_mcp_server.py`.

Por último, revisamos el comportamiento de los distintos elementos de MCP. Se comprobó que las herramientas definidas con @mcp.tool() pueden ser invocadas por Claude cuando el usuario solicita una acción, como listar correos o enviar un email. Sin embargo, los recursos definidos con @mcp.resource() no parecen ser legíbles para Claude de forma automática. Por este motivo, se tuvo que invocar manualmente el recurso implementado (en este caso, el recurso encargado de la obtención del perfil de Gmail).


## 5. Posibles mejoras o extensiones

El servidor actual cumple los requisitos principales de la practica, pero podria mejorarse y ampliarse con nuevas funcionalidades:

- Crear borradores antes de enviar los correos para evitar enviar directamente los mensajes generados por el modelo.
- Anadir una tool para buscar correos aplicando filtros como remitente, asunto, fecha o palabras clave.
- Permitir responder a un hilo existente en lugar de enviar siempre un correo nuevo.
- Permitir la gestión de etiquetas, archivado o marcado como leido/no leido.
- Validar parametros de entrada (por ejemplo, comprobando que el destinatario tenga formato de email).

En conclusión, el servidor implementado demuestra cómo MCP puede conectar un modelo conversacional con un servicio externo real, integrando herramientas, recursos y prompts sobre una API protegida mediante OAuth. Esto permite que el modelo no se limite únicamente a generar texto, sino que pueda interactuar de forma segura y controlada con servicios externos, como Gmail en este caso, para consultar información o ejecutar acciones concretas.