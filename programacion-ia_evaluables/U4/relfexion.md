# Reflexión.md

## ¿Qué caso elegiste y por qué? 
He realizado los casos 2 y 3 aunque me hubiera gustado hacer todos.
Me parecen interesantes y me costaba decidirme por uno solo.


## ¿Qué dificultades encontraste? 

### Dificultades con el caso 2
En la realización de este caso no ha sugrido dificultad alguna.

### Dificultades con el caso 3
Durante la realización del caso 3 he tenido un problema.

Ocurrió en la configuración del agente en el workflow de n8n. Se me advertía de un error en la configuración del campo "Prompt (User Message)" aunque esta era correcta.

Para solucionarlo, le pedí al AI Companion de N8N que revisara el workflow y corrigiera los errores. Aunque el companion no modificó la configuración, el error desapareció tras la revisión de la IA. Imagino que fue un error interno de N8N.


## ¿Qué mejorarías con más tiempo? 
Cambiaría bastantes cosas.

Lo primero sería limitar el acceso al agente. Actualmente el bot de telegram es público y cualquier persona le puede enviar mensajes e interactuar con él. Esto se convierte en un problema el caso 2, donde el agente interactúa con Gmail y podría utilizarse para el envío de spam o correos maliciosos.
Eso se puede cambiar fácil en la configuración del nodo de telegram, que permite restringir el trigger a un chatid o userid específico.

También tengo un pequeño fallo en la gestión de la memoria del agente. 
El agente guarda las conversaciones en tablas, diferencias por contener el "message.chat.username" (que es el nombre de usuario de la persona que le envía un mensaje por telegram) en el nombre de la tabla. 
Si se diese el caso en el que la persona que tiene una conversación previa, se cambia el nombre de usuario de telegram, el agente crearía una nueva tabla por lo que el bot perdería la conversación previa.
Lo lógico, sería usar el "message.chat.id" el lugar del "message.chat.username", porque el message.chat.id es un identificador único para cada chat. Pero no quiero porque tendría más problemas para identificar qué tabla pertenece a qué usuario, algo que me viene bien para monitorizar los usuarios que interactúan con el bot.

## ¿Cómo lo aplicarías en un contexto profesional real?
El caso 1 se podría usar directamente como un bot de asistencia al cliente de una tienda, tanto online como presencial. Sería capaz de resolver preguntas sobre los productos (stock, precios, descuentos...), sobre las políticas del comercio (devoluciónes, envios, garantías).
También se podría utilizar para ver qué es lo que buscan los clientes o de qué se quejan con mas frecuencia.
Incluso podrían aconsejar a los clientes sobre qué articulo es más apropiado para su situación.

El caso 2 se podría usar como un secretario digital que ayude con la redacción de correos. Al ser capaz el bot de generar el asunto y el cuerpo del correo, podría ahorrar mucho tiempo de escritura y corrección de errores.

El caso 3 se podría utilizar como una herramienta de consulta rápida y sintetizada de información.

Es importante destacar que, debido a la simplicidad de la estructura de los casos (especialmente de los casos 2 y 3), existen numerosas limitaciones en su utilización.