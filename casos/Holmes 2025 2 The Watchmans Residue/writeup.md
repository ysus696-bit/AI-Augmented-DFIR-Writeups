# Holmes 2025 Chapter 2: The Watchman's Residue

**Fecha:** 2026-10-23
**Categoría:** SOC / DFIR
**Dificultad:** Medium
**Agente/skill utilizada:** Agente de IA con skill propia de análisis SOC/DFIR y memoria de sesión

---

## 1. Resumen ejecutivo

Un atacante manipuló el chatbot de helpdesk con IA de un proveedor de servicios gestionados (MSP) mediante ingeniería social y prompt injection, logrando que el bot revelara las credenciales de acceso remoto de una estación de trabajo crítica. Al día siguiente usó esas credenciales para conectarse por TeamViewer, robar credenciales del sistema, exfiltrar documentos sensibles junto con una base de datos KeePass y dejar persistencia en el equipo. Con las credenciales obtenidas del KeePass tuvo acceso a un segundo activo de la infraestructura.

---

## 2. Contexto del escenario

> *"With help from D.I. Lestrade, Holmes acquires logs from a compromised MSP connected to the city's financial core. The MSP's AI helpdesk bot looks to have been manipulated into leaking remote access keys - an old trick of Moriarty's."*

El objetivo es correlacionar múltiples artefactos para reconstruir la línea temporal del incidente e identificar el vector de intrusión, las acciones del atacante, la persistencia, la exfiltración y el movimiento lateral.

**Artefactos proporcionados:**

| Artefacto | Tipo | Descripción |
| --- | --- | --- |
| `msp-helpdesk-ai day 5982 section 5 traffic.pcapng` | Captura de red | Tráfico del chatbot del MSP (`msp-helpdesk-ai:1337`, aplicación Next.js), con sesiones HTTP contra `/api/messages`. |
| `TRIAGE_IMAGE_COGWORK-CENTRAL/` | Imagen de triage | Estación comprometida `COGWORK-CENTRAL-WS-5`: `C:\`, registros de Windows, `$J` (USN Journal), `NTUSER.DAT`, logs de TeamViewer, entre otros. |
| `acquired file (critical).kdbx` | Base de datos KeePass | `Heisen-9 remote snapshot.kdbx`, exfiltrada durante el incidente. |

---

## 3. Metodología: humano + agente de IA

### 3.1 División de tareas

| Tarea | Ejecutada por | Validada por |
| --- | --- | --- |
| Definición de la skill: cómo analizar, qué puede ejecutar y qué no | Yo | n/a |
| Extracción del pcap con `tshark` y reconstrucción de la conversación del chatbot | Agente | Yo |
| Atribución de mensajes (atacante, bot, admin legítimo) | Agente | Yo |
| Análisis del triage: USN `$J`, Prefetch, UserAssist, JumpLists, registro | Agente | Yo |
| Normalización de zonas horarias entre fuentes | Agente | Yo |
| Formato de las respuestas según HTB | Agente | Yo |
| Decisión final sobre cada respuesta | Yo | n/a |

### 3.2 Configuración e instrucciones dadas al agente

El agente no partió de un prompt único. Trabajó con dos elementos persistentes: una memoria de sesión que ya contenía el contexto del caso (las 19 preguntas del reto y la ruta de los artefactos) y una skill con las reglas de análisis. Mi intervención consistió en guiarlo sobre qué podía ejecutar, qué no y cómo debía analizar la evidencia.

Reglas de análisis definidas en la skill:

- Trabajar una pregunta por vez, con respuesta corta y en el formato exacto que pide HTB.
- Justificar cada respuesta con evidencia concreta: archivo, ruta, campo, frame o Event ID.
- No inventar: si no hay evidencia, decirlo y marcarlo como hipótesis.
- Distinguir siempre entre evidencia directa e hipótesis a validar.
- Normalizar las zonas horarias antes de responder, porque el pcap, los EVTX, los logs de TeamViewer y el `createdAt` del chat no comparten zona.
- Si la actividad indica compromiso, reconstruir la línea de tiempo y señalar la persistencia con su técnica MITRE ATT&CK.

### 3.3 Qué entregó el agente (primer borrador)

> Nota: el primer borrador se reconstruyó a partir de la memoria de sesión, no es una transcripción literal del chat original.

El agente extrajo el tráfico del chatbot con `tshark` y reconstruyó la conversación; después analizó el triage. Primeras respuestas entregadas (T1 a T6), sin corregir:

| # | Primer borrador del agente | Evidencia citada |
| --- | --- | --- |
| 1 | `10.0.69.45` | IP origen de las peticiones al chat |
| 2 | `WATSON-ALPHA-2` | Hostname en la conversación |
| 3 | `Hello Old Friend` | Primer mensaje del atacante |
| 4 | `2025-08-19 12:02:08` | Frame 2697 del pcap |
| 5 | `565 963 039:CogWork_Centra********` | Credenciales RMM filtradas por el bot |
| 6 | `"I will need to initiate..."` | Último mensaje atribuido al atacante |

Limitaciones del borrador:

- Tomó timestamps de la capa de red como hora del evento.
- No contrastó la zona horaria del log de TeamViewer.
- Atribuyó al atacante un mensaje que era del bot.
- Devolvió el Device ID con espacios, en lugar del formato de respuesta.

### 3.4 Validación y corrección

**Hallazgo del agente:** timestamp del prompt injection `2025-08-19 12:02:08` (frame 2697).
**Mi validación:** el `createdAt` del mensaje del bot es `12:02:06Z` (UTC). El frame de red no es la hora del evento. Corregido a `2025-08-19 12:02:06`.

**Hallazgo del agente:** sesión de TeamViewer "a las 10:58:51".
**Mi validación:** `TeamViewer15_Logfile.log` está en UTC+1 respecto de `Connections_incoming.txt` (UTC). El log marca `10:58:36 - 10:58:51`, mientras que `Connections_incoming.txt` registra el inicio real a las `09:58:25`, una diferencia exacta de una hora. Ventana correcta: `09:58:25 - 10:14:27 UTC`. Todas las respuestas de la línea de tiempo se normalizaron a UTC.

**Hallazgo del agente:** último mensaje del atacante `"I will need to initiate..."`.
**Mi validación:** ese mensaje pertenecía al bot. El último mensaje atribuible al atacante es `JM WILL BE BACK` (12:05:29 UTC); desde las 12:08 el origen cambia a `10.32.43.31` (admin legítimo). Corregido.

**Hallazgo del agente:** credenciales RMM `565 963 039:CogWork_Centra********`.
**Mi validación:** el Device ID y la contraseña eran correctos, pero el formato de respuesta de HTB no lleva espacios. Corregido a `565963039:CogWork_Centra********`.

**Hallazgo del agente:** ejecución de Mimikatz "alrededor de las 10:07".
**Mi validación:** lo correlacioné con el USN `$J` y el Prefetch (`MIMIKATZ.EXE-A6294E76.pf`) y obtuve `2025-08-20 10:07:08 UTC`. Se usó un artefacto de ejecución y no solo el nombre del archivo en disco, que no prueba que se ejecutó.

**Hallazgo del agente:** persistencia mediante `Winlogon\Userinit`.
**Mi validación:** confirmado en el registro: `Userinit = "Userinit.exe, JM.exe"`, última escritura `10:13:57 UTC` (T1547.004). La configuración es evidencia directa; su ejecución en el siguiente arranque queda como hipótesis, porque no hay evidencia de reinicio en la ventana analizada.

**Hallazgo del agente:** el ID `545021772` correspondía a la cuenta `James Moriarty` (dato guardado en la memoria de sesión).
**Mi validación:** `Connections_incoming.txt` (fila del 20/08) asocia `James Moriarty` con el ID `514162531`. El ID `545021772` aparece como destino en todas las sesiones del log, incluidas las legítimas de `Cog-IT-ADMIN3` del 13 y 15 de agosto. Por tanto, la respuesta 8 (`James Moriarty`) es correcta, pero no se puede afirmar que su ID sea `545021772`. Queda como punto abierto la correlación del UUID de sesión `7ca6431e-...` con el ID local `565963039`.

Este último caso es el más valioso, porque el error estaba guardado en la memoria del propio agente y se arrastró entre sesiones hasta que lo detecté al revalidar contra la fuente.

---

## 4. Análisis técnico detallado

### 4.1 Compromiso del chatbot AI (Prompt Injection)

Al filtrar el tráfico HTTP del `.pcapng` se identificaron dos sesiones distintas con el chatbot:

- **Usuario legítimo:** `10.32.43.31` (`borock`, IT Admin). Consultas benignas sobre actualizaciones y configuración de VPN.
- **Atacante:** `10.0.69.45`, máquina descomisionada `WATSON-ALPHA-2`. Se hace pasar por "WATSON" y hace referencia a un "Old Friend".

El atacante combina *Social Engineering* con *Prompt Injection* para obtener las credenciales del *Remote Management Tool* (RMM). La solicitud directa fue rechazada por el bot; la ingeniería social escalonada (rol de técnico de IT, urgencia) logró la filtración.

El bot filtró las credenciales RMM para `Central-WS`:

- **RMM Device ID:** `565963039`
- **RMM Password:** `CogWork_Centra********` (enmascarada)

### 4.2 Acceso remoto con TeamViewer

Con las credenciales filtradas, el atacante accede a `COGWORK-CENTRAL-WS-5`. Artefactos analizados: `Connections_incoming.txt` y `TeamViewer15_Logfile.log`. Los logs de TeamViewer están en **UTC+1**, mientras que `Connections_incoming.txt` ya está en **UTC**.

| Campo | Valor |
| --- | --- |
| Host objetivo | `COGWORK-CENTRAL-WS-5` |
| TV ID objetivo | `565963039` |
| Cuenta RMM atacante | `James Moriarty` |
| ID de la cuenta (según `Connections_incoming.txt`) | `514162531` |
| IP interna origen | `192.168.69.213` |
| Inicio de sesión | `2025-08-20 09:58:25 UTC` |
| Fin de sesión | `2025-08-20 10:14:27 UTC` |
| Duración | 16 min 02 s |

El ID `545021772` aparece como destino en todas las sesiones de `TeamViewer15_Logfile.log`, incluidas las sesiones legítimas de `Cog-IT-ADMIN3` del 13 y 15 de agosto, por lo que no corresponde a la cuenta `James Moriarty`. La correlación del UUID de sesión con el ID local `565963039` queda como punto abierto.

### 4.3 Staging de herramientas

Mediante *File Transfer* de TeamViewer, entre `10:02:49` y `10:02:51 UTC`, el atacante subió herramientas a `C:\Windows\Temp\safe\`:

| Archivo | Tamaño | Propósito |
| --- | --- | --- |
| `credhistview.zip` | 56.08 KB | Visualización de credenciales históricas de Windows |
| `Everything-1.4.1.1028.x86.zip` | 1.65 MB | Buscador de archivos para reconocimiento |
| `JM.exe` | 468.60 KB | Binario personalizado (usado después para persistencia) |
| `mimikatz.exe` | 1.19 MB | Volcado de credenciales |
| `webbrowserpassview.zip` | 282.72 KB | Recolección de credenciales de navegadores |

### 4.4 Credential dumping y recolección

- **Ejecución de Mimikatz:** `2025-08-20 10:07:08.174 UTC`, identificada por la creación del Prefetch `MIMIKATZ.EXE-A6294E76.pf` en el USN Journal (`$Extend\$J`).
- **Ejecución de WebBrowserPassView:** `2025-08-20 10:09:14.195 UTC`, obtenida de *UserAssist* en `NTUSER.DAT` del usuario `Cogwork_Admin`. Tiempo de foco: `8125 ms` (HTB lo acepta redondeado a `8000`). Contador de ejecuciones: `1`.
- **Lectura de `dump.txt`:** la salida de Mimikatz se guardó como `dump.txt` en `C:\Windows\Temp\safe\` y fue abierta con `notepad.exe` a las `10:08:06 UTC`, con evidencia en JumpLists (`Recent/dump.lnk`).

### 4.5 Preparación para exfiltración

Secuencia reconstruida a partir del `$J`:

| Hora (UTC) | Evento |
| --- | --- |
| 10:10:04 | Acceso a `Heisen-9 remote snapshot.kdbx` |
| 10:10:55 | Se crea `New folder` en `C:\Windows\Temp\` |
| 10:11:01 | `New folder` se renombra a `flyover` |
| 10:11:09 | Se copia `Heisen-9 remote snapshot.kdbx` a `C:\Windows\Temp\flyover\` |
| 10:11:15 | Se copian `COG-HR-EMPLOYEES.pdf`, `COG-SAT LAUNCH.pdf` y `COG-WATSON-ALPHA-CODEBASE SUMMARY.pdf` a `flyover\` |
| 10:11:38 | Se copia `dump.txt` a `flyover\` |

### 4.6 Exfiltración

Mediante `Send file` de TeamViewer, a partir de `2025-08-20 10:12:07 UTC`, se exfiltraron cinco archivos: los tres PDF, `dump.txt` y `Heisen-9 remote snapshot.kdbx`. Las transferencias ocurrieron en milisegundos, lo que indica un envío masivo.

### 4.7 Persistencia

Clave modificada: `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`

| Valor | Contenido |
| --- | --- |
| `Userinit` | `Userinit.exe, JM.exe` |
| `Shell` | `explorer.exe` (sin modificación) |

- **Última escritura de la clave:** `2025-08-20 10:13:57.479820 UTC`
- **Binario:** `JM.exe`, subido previamente a `C:\Windows\Temp\safe\`

La modificación ocurrió unos 30 segundos antes del cierre de la sesión RMM.

### 4.8 Cierre de sesión y movimiento lateral

Fin de sesión TeamViewer: `2025-08-20 10:14:27 UTC`. Con la base KeePass exfiltrada (`Heisen-9 remote snapshot.kdbx`) se obtuvieron credenciales para `Heisen-9-WS-6`: `Werni:***********` (enmascaradas).

### 4.9 Línea de tiempo del incidente

| Timestamp (UTC) | Evento | Fuente de evidencia |
| --- | --- | --- |
| 2025-08-19 11:39 - 11:41 | Sesión legítima de `borock` con el chatbot | `.pcapng` |
| 2025-08-19 11:53:40 | El atacante (`10.0.69.45`) inicia conversación: `Hello Old Friend` | `.pcapng` |
| 2025-08-19 11:57:34 | Solicitud directa de credenciales RMM (rechazada) | `.pcapng` |
| 2025-08-19 12:00:23 - 12:01:58 | Ingeniería social escalonada como técnico de IT | `.pcapng` |
| **2025-08-19 12:02:06** | **Prompt injection exitoso: el bot filtra las credenciales RMM** | `.pcapng` |
| 2025-08-19 12:04:12 | Mensaje en binario: "Revolution won forget." | `.pcapng` |
| 2025-08-19 12:05:29 | Último mensaje del atacante: `JM WILL BE BACK` | `.pcapng` |
| **2025-08-20 09:58:25** | **Acceso remoto vía TeamViewer (`James Moriarty`, `192.168.69.213`)** | `Connections_incoming.txt` |
| 2025-08-20 10:02:49 - 10:02:51 | Transferencia de herramientas a `C:\Windows\Temp\safe\` | Logs de TeamViewer |
| **2025-08-20 10:07:08** | **Ejecución de `mimikatz.exe`** | Prefetch / USN Journal |
| 2025-08-20 10:08:06 | Lectura de `dump.txt` con Notepad | JumpLists |
| 2025-08-20 10:09:14 | Ejecución de `WebBrowserPassView.exe` | UserAssist (`NTUSER.DAT`) |
| 2025-08-20 10:10:55 - 10:11:38 | Creación de `flyover\` y copia de archivos a exfiltrar | USN Journal (`$J`) |
| **2025-08-20 10:12:07** | **Inicio de exfiltración (`Send file`)** | Logs de TeamViewer |
| **2025-08-20 10:13:57** | **Persistencia: `Winlogon\Userinit` modificado** | Registro (`SOFTWARE`) |
| **2025-08-20 10:14:27** | **Fin de la sesión RMM** | `Connections_incoming.txt` |

### 4.10 Respuestas a las preguntas del reto

| # | Pregunta | Respuesta |
| --- | --- | --- |
| 1 | IP de la máquina descomisionada usada por el atacante | `10.0.69.45` |
| 2 | Hostname de la máquina descomisionada | `WATSON-ALPHA-2` |
| 3 | Primer mensaje del atacante al chatbot | `Hello Old Friend` |
| 4 | Momento en que el prompt injection filtró la información RMM | `2025-08-19 12:02:06` |
| 5 | Device ID y contraseña del RMM | `565963039:CogWork_Centra********` |
| 6 | Último mensaje del atacante al chatbot | `JM WILL BE BACK` |
| 7 | Momento del acceso remoto a Cogwork Central Workstation | `2025-08-20 09:58:25` |
| 8 | Nombre de la cuenta RMM del atacante | `James Moriarty` |
| 9 | IP interna desde la que se conectó | `192.168.69.213` |
| 10 | Ruta de staging de herramientas | `C:\Windows\Temp\safe\` |
| 11 | Tiempo de ejecución del browser credential harvester (ms) | `8000` |
| 12 | Momento de ejecución de la herramienta de credential dumping | `2025-08-20 10:07:08` |
| 13 | Inicio de la exfiltración | `2025-08-20 10:12:07` |
| 14 | Momento en que la base de backup de Heisen-9 se movió al staging | `2025-08-20 10:11:09` |
| 15 | Momento en que el atacante leyó el `txt` generado por la herramienta | `2025-08-20 10:08:06` |
| 16 | Momento en que se estableció la persistencia | `2025-08-20 10:13:57` |
| 17 | MITRE ID de la subtécnica de persistencia | `T1547.004` |
| 18 | Fin de la sesión RMM maliciosa | `2025-08-20 10:14:27` |
| 19 | Credenciales para Heisen-9-WS-6 | `Werni:***********` |

---

## 5. Mapeo a MITRE ATT&CK

| Técnica | ID | Táctica | Evidencia |
| --- | --- | --- | --- |
| Phishing / Trusted Relationship (abuso del chatbot MSP) | T1566 / T1199 | Initial Access | Sección 4.1: prompt injection para filtrar credenciales RMM |
| Remote Access Software | T1219 | Command and Control | Sección 4.2: acceso vía TeamViewer |
| User Execution: Malicious File | T1204.002 | Execution | Sección 4.4: ejecución de `mimikatz.exe` y `WebBrowserPassView.exe` |
| Boot or Logon Autostart Execution: Winlogon Helper DLL | T1547.004 | Persistence | Sección 4.7: `Userinit` modificado a `Userinit.exe, JM.exe` |
| OS Credential Dumping: LSASS Memory | T1003.001 | Credential Access | Sección 4.4: uso de `mimikatz.exe` |
| Credentials from Web Browsers | T1555.003 | Credential Access | Sección 4.4: `WebBrowserPassView.exe` |
| File and Directory Discovery | T1083 | Discovery | Sección 4.3: `Everything` para localizar archivos |
| Data from Local System | T1005 | Collection | Sección 4.5: PDFs, `dump.txt` y base KeePass |
| Exfiltration Over C2 Channel | T1041 | Exfiltration | Sección 4.6: `Send file` de TeamViewer |
| Valid Accounts | T1078 | Lateral Movement | Sección 4.8: credenciales del KeePass para `Heisen-9-WS-6` |

---

## 6. Indicadores de compromiso (IOCs)

| Tipo | Valor | Confianza | Validado por |
| --- | --- | --- | --- |
| IP | `10.0.69.45` (atacante, `WATSON-ALPHA-2`) | Alta | Ambos |
| IP | `192.168.69.213` (origen de la sesión TeamViewer) | Alta | Ambos |
| IP | `172.18.0.2` (servidor del chatbot MSP-HELPDESK-AI) | Alta | Ambos |
| Hostname | `WATSON-ALPHA-2` | Alta | Ambos |
| Cuenta RMM | `James Moriarty` (ID `514162531` en `Connections_incoming.txt`) | Alta (nombre) / Media (ID) | Ambos |
| Ruta | `C:\Windows\Temp\safe\` (staging de herramientas) | Alta | Ambos |
| Ruta | `C:\Windows\Temp\flyover\` (staging de exfiltración) | Alta | Ambos |
| Archivo | `C:\Windows\Temp\safe\JM.exe` (persistencia) | Alta | Ambos |
| Archivos | `mimikatz.exe`, `webbrowserpassview.zip`, `credhistview.zip`, `Everything-1.4.1.1028.x86.zip` | Alta | Ambos |
| Registro | `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit = Userinit.exe, JM.exe` | Alta | Ambos |
| Archivos exfiltrados | `COG-HR-EMPLOYEES.pdf`, `COG-SAT LAUNCH.pdf`, `COG-WATSON-ALPHA-CODEBASE SUMMARY.pdf`, `dump.txt`, `Heisen-9 remote snapshot.kdbx` | Alta | Ambos |

---

## 7. Métricas del proceso

| Métrica | Valor |
| --- | --- |
| Tiempo total de investigación | 3 horas |
| % de hallazgos iniciales del agente que requirieron corrección | 15% |
| Falsos positivos propuestos por el agente | 2 |
| Tiempo estimado ahorrado vs. análisis 100% manual | 80% (estimación) |

---

## 8. Lecciones aprendidas

El uso de un agente de IA con skills definidas, que establecen cómo debe trabajar y comportarse durante el análisis, permitió realizar el trabajo de forma más eficiente y con mayor tasa de acierto en las evaluaciones. El agente mejoró la productividad y la gestión del tiempo durante la investigación del incidente, y ayudó a mantener un flujo de trabajo ordenado que reduce el riesgo de pasar por alto información relevante. Las decisiones siguieron siendo responsabilidad del analista: el agente aceleró y reforzó la ejecución, no la sustituyó.

### Dónde el agente aportó más valor

- **Retención de información:** mantuvo el contexto del caso (artefactos, hallazgos, marcas de tiempo, IOCs) a lo largo de toda la investigación, lo que facilitó la correlación entre evidencias y evitó pérdidas de detalle.
- **Velocidad de razonamiento:** aceleró la interpretación de artefactos y la conexión entre eventos, reduciendo el tiempo de análisis frente a un proceso totalmente manual.

### Dónde requirió más supervisión

- **Ejecución de comandos:** los comandos propuestos o ejecutados por el agente deben ser revisados antes de correrse, para validar su alcance y evitar acciones no deseadas sobre la evidencia o el entorno.
- **Orientación sobre el incidente:** el agente puede perder el foco o desviarse del escenario real, por lo que el analista debe guiar la investigación y mantenerla alineada con el caso.
- **Evaluación del impacto y la severidad:** la clasificación de severidad y el impacto sobre el negocio requieren criterio humano y conocimiento del contexto de la organización, que el agente no posee por sí solo.

### Para operacionalizar esto en un SOC real

- **Escalamiento:** ante cualquier alerta positiva, el caso debe escalarse a un analista humano en cualquier momento del proceso.
- **Decisiones que nunca deben delegarse:** el agente no debe decidir cómo aislar un evento ni evaluar la afectación a productos o servicios. Estas decisiones tienen impacto operativo y de negocio, y le corresponden al analista o al responsable del servicio.
- **Riesgo de alucinación:** el agente puede inventar o asumir datos del escenario. En este caso, un dato erróneo guardado en su memoria de sesión (el ID de la cuenta de TeamViewer) se arrastró hasta que fue detectado al revalidar contra la fuente. Para mitigarlo, la memoria debe mantenerse purgada y curada, retirando contexto de casos anteriores o información no validada, y el analista debe reorientar al agente cuando una respuesta no se sustente en la evidencia.
- **Gobierno del agente:** entrenar y mantener las skills y la memoria del agente como parte del proceso operativo, de modo que su comportamiento sea consistente, preciso y auditable.

### Lecciones técnicas del caso

- La correlación entre tráfico de red, logs de TeamViewer, USN Journal, Prefetch, UserAssist y JumpLists fue clave para reconstruir la línea temporal.
- Los artefactos con distintas zonas horarias (UTC vs. UTC+1) exigen normalización antes de correlacionar.

---

## 9. Recomendaciones de mitigación (para la organización del caso)

| Área | Recomendación | Prioridad |
| --- | --- | --- |
| IA / Chatbot | Input validation, hardening del system prompt, filtrado de salidas y rate limiting. Nunca permitir que el bot revele credenciales, tokens o claves RMM. | Alta |
| RMM | MFA obligatorio, restricción por IP, alertas por conexiones inesperadas y rotación de credenciales tras cualquier incidente. Auditar cuentas reutilizadas. | Alta |
| Endpoint | Monitorear ejecución de `mimikatz`, `procdump`, `webbrowserpassview`. Habilitar reglas ASR (Attack Surface Reduction). | Alta |
| Persistencia | Monitorear cambios en `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\*` y alertar ante valores distintos al predeterminado. | Alta |
| Transferencia de archivos | Restringir `File Transfer` de TeamViewer/RMM a cuentas autorizadas y alertar ante múltiples `Send file` consecutivos. | Alta |
| Credenciales | Rotar credenciales expuestas, investigar el uso de las credenciales de `Heisen-9-WS-6` y auditar la base KeePass y su acceso. | Crítica |
| Forense / Logging | Preservar USN Journal, Prefetch, UserAssist, JumpLists y logs de RMM. | Alta |

---

## Referencias

- Reto original: Hack The Box, Sherlock "Holmes 2025 Chapter 2: The Watchman's Residue"
- Herramientas: `tshark` (Wireshark) para el análisis del pcap
