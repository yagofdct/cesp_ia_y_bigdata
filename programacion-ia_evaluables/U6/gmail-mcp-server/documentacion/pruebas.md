# 7 Pruebas
Se han realizado las siguientes pruebas en Claude Desktop para verificar que todos los componentes funcionan correctamente. 
Se documenta cada prueba con capturas de pantalla ubicadas en gmail-mcp-server/capturas/.


## 7.1 Prueba de Listar Emails
Entrada en Claude Desktop:

Lista mis últimos 5 emails

Resultado: Claude utiliza la herramienta list_emails y muestra los emails recientes con remitente, asunto y fecha.

## 7.2 Prueba de Enviar Email
Entrada en Claude Desktop:

Envía un email de prueba a mi.email@ejemplo.com con asunto "Test MCP"
y cuerpo "Este es un email enviado desde mi servidor MCP"

Resultado: Claude utiliza la herramienta send_email y confirma el envío con el ID del mensaje.

## 7.3 Prueba de Consultar Perfil
Entrada en Claude Desktop:

¿Cuál es mi perfil de Gmail?

Resultado: Claude accede al recurso gmail://profile y muestra la dirección de email, el número total de mensajes y de hilos.

## 7.4 Prueba del Prompt de Redacción
Uso del prompt de redacción desde la interfaz de Claude Desktop par la redacción de un correo con las siguientes características:

Destinatario: Juan García
Tema: Reunión del próximo lunes
Tono: Profesional

Resultado: Claude utiliza el prompt redactar_email para generar un borrador de email con el tono y contenido solicitados.


## 7.5 Documentación de las Pruebas

Para cada prueba se incluyen entre una y tres capturas, en las que se muestra:

- La pregunta o instrucción introducida en Claude Desktop.
- La respuesta generada por Claude, junto con la indicación de la herramienta o recurso utilizado.
- El resultado final de la acción realizada.

Las capturas cuyo nombre termina en “claude” corresponden a Claude Desktop, mientras que las que terminan en “gmail” muestran el resultado obtenido en Gmail.