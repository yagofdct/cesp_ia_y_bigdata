# 1. ¿En cuál de las 4 etapas viste más claramente la ventaja de LangGraph sobre una Chain LCEL?
La ventaja de LangGraph sobre una Chain LCEL se ve mucho más clara en la Etapa 4, Agente ReAct con tool personalizada.
En las primeras etapas, una Chain LCEL puede resolver bastante bien el flujo porque son casos más lineales. Sin embargo, en la Etapa 4 el flujo deja de ser lineal, ya que el agente debe decidir si necesita usar una herramienta, elegir cuál usar, ejecutar esa herramienta, incorporar el resultado al estado y generar la respuesta final.
La Etapa 3 también muestra ventajas, porque el flujo deja de ser lineal y aparece el enrutado condicional. Aun así, la diferencia más clara sigue estando en la Etapa 4, porque además del enrutado aparecen herramientas, pasos intermedios, memoria y toma de decisiones dinámica.

# 2. ¿Cuál fue el bug o malentendido más difícil de depurar mientras hacías los ejercicios?
Lo más difícil fue trabajar con la API gratuita de OpenRouter. Al usar un modelo gratuito, me encontré con límites de uso entre peticiones, lo que podía provocar errores o respuestas incompletas si ejecutaba el código varias veces seguidas.
Esto fue confuso porque el problema no estaba en el código, sino en las restricciones externas del proveedor del modelo. Por ese motivo, para comprobar que el código funcionaba correctamente tuve que ejecutarlo por partes y dejar pasar tiempo entre ejecuciones para evitar que fallase el flujo del agente.

# 3. Diseña (sin programarlo) un grafo para una aplicación real de tu interés. Describe los nodos, las aristas y el estado en 5-10 líneas.
Diseñaría un grafo para sincronizar tickets de una query de Jira con una hoja de Google Sheets llamada “Backlog Tickets”.
* El estado incluiría: tickets obtenidos, ticket actual, existencia en backlog, tipo de ticket, tecnología detectada, motivo_clasificacion_tecnologia, tecnologías permitidas, acción realizada, usuario_a_notificar y logs.
* Los nodos serían: obtener_tickets_jira, consultar_backlog, comprobar_existencia, actualizar_ticket, clasificar_ticket, validar_tecnologia, añadir_ticket_backlog, notificar_ticket_permitido, notificar_tecnologia_no_permitida, registrar_log y siguiente_ticket_o_fin.
* Las aristas principales serían: START → obtener_tickets_jira → consultar_backlog → comprobar_existencia.

## 3.1 Desglose del funcionamiento
El flujo empezaría obteniendo los tickets desde Jira y recorriéndolos uno a uno. Si un ticket ya existe en la hoja “Backlog JIRA”, pasaría por actualizar_ticket para modificar los campos que hayan cambiado y después por registrar_log.
- Si el ticket no existe, pasaría por clasificar_ticket, donde se detectaría la tecnología relacionada y se guardaría también el motivo de esa clasificación.
- Desde comprobar_existencia habría una arista condicional: si el ticket existe → actualizar_ticket; si no existe → clasificar_ticket.
Después, validar_tecnologia comprobaría si la tecnología pertenece a la lista de tecnologías permitidas.
- Si la tecnología está permitida, el ticket se añadiría a la hoja mediante añadir_ticket_backlog y después se ejecutaría notificar_ticket_permitido, enviando a un usuario la información del ticket, la tecnología elegida y el motivo por el cual se ha elegido dicha tecnología.
- Si la tecnología no está permitida, el grafo ejecutaría notificar_tecnologia_no_permitida, avisando de la existencia del ticket y de la tecnología detectada.
- Desde validar_tecnologia habría una arista condicional: si la tecnología está permitida → añadir_ticket_backlog → notificar_ticket_permitido; si no está permitida → notificar_tecnologia_no_permitida.
Todas las ramas terminarían en registrar_log, guardando en la hoja “Logs agente” la acción realizada: actualización, alta de nuevo ticket, notificación de ticket permitido, notificación de tecnología no permitida o error.
* Finalmente, registrar_log conectaría con siguiente_ticket_o_fin. Si quedan tickets por procesar, el grafo volvería a consultar_backlog; si no quedan tickets, terminaría en END.
