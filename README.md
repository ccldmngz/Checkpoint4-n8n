# Checkpoint 4 - Sincronización del Cerebro Agéntico con Ecosistemas de Negocio

**Autora:** Cecilia Dominguez

Workflow en n8n que conecta un agente de IA con tres herramientas reales de la organización mediante OAuth2:

- **HubSpot** (CRM de la tienda)
- **Gmail** (casilla de soporte al cliente)
- **Slack** (canal del equipo de operaciones)

El proyecto parte del workflow del Módulo 3 (con memoria y contexto por Session_ID) y le suma las integraciones y los controles preventivos de este checkpoint.

## Flujo

```
Gmail Trigger
   -> IF Auto-reply (descarta correos automáticos)
   -> Normalizar Gmail
   -> Buscar / crear usuario en MEMORIA
   -> AI Agent (clasifica y redacta)
   -> IF Viene de Gmail?
   -> Limpiar Payload
   -> Look up Contacto (HubSpot)
   -> IF CRM
        - true:  Update Contacto
        - false: Create Contacto
   -> Create Draft (Gmail, aprobación humana)
   -> Payload Slack (Set)
   -> Aviso a Slack
```

## Los 4 nodos clave

1. **IF anti auto-reply:** ubicado justo después del trigger de Gmail. Descarta correos cuyo asunto contenga *Auto-reply*, *Out of office* o *Undeliverable*, y remitentes `no-reply@` / `noreply@`. Corta el bucle infinito de auto-respuestas.
2. **Look up Contacto + IF CRM:** busca el contacto en HubSpot por email antes de crearlo. Si existe se actualiza; si no, se crea. Previene el Error 409 (contactos duplicados).
3. **Create Draft:** la respuesta generada por la IA se guarda únicamente como borrador en Gmail. No hay ningún nodo de envío, por lo que siempre hay una persona que revisa y aprueba antes de enviar (Human-in-the-loop).
4. **Set de limpieza de payload (Limpiar Payload y Payload Slack):** deja solo los campos necesarios (email, nombre, asunto, respuesta, categoría) y verifica que el email no esté vacío. Evita el Error 400 por payload mal formado y no satura el canal de Slack con datos pesados.

## Principio de mínimo privilegio

| Herramienta | Permisos usados |
|---|---|
| Gmail | Lectura de mensajes y creación de borradores (sin envío) |
| HubSpot | Lectura y escritura de contactos únicamente |
| Slack | Solo `chat:write` |

## Credenciales

Las tres conexiones están activas en n8n mediante OAuth2:

![Credentials de n8n](credentials.png)

## Pruebas realizadas (test de regresión manual)

| Caso | Resultado esperado | Resultado |
|---|---|---|
| Cliente nuevo | Contacto creado en HubSpot, borrador en Gmail, aviso en Slack | OK |
| Cliente existente | IF CRM por true (Update), sin duplicados en HubSpot | OK |
| Asunto "Out of office" | El flujo se corta en el IF anti auto-reply | OK |
| Mensaje por Telegram | La rama de Telegram sigue funcionando | OK |

## Cómo usarlo

1. Descargar `checkpoint4_cecilia_dominguez.json`.
2. En n8n: **Workflow > Import from File**.
3. Asignar las credenciales propias de Gmail, HubSpot, Slack y el modelo de IA.
4. Configurar la hoja MEMORIA y el canal de Slack.
5. Ejecutar con **Execute workflow** y enviar un correo de prueba a la casilla conectada.
