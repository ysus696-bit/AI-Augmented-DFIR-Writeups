---
name: threat-hunting-proactivo
description: Construye y ejecuta una hipótesis de threat hunting sobre Wazuh/SIEM (no reactiva a una alerta existente), basada en TTPs de MITRE ATT&CK relevantes para el entorno. Úsala cuando Andres pida "buscar algo que no haya saltado como alerta" o quiera validar si una técnica específica ya ocurrió sin ser detectada.
---

# Threat Hunting Proactivo

## Objetivo
Buscar actividad maliciosa que no generó una alerta automática, partiendo de
una hipótesis concreta en vez de "revisar logs a ver qué se encuentra".

## Cuándo se activa
- Petición explícita de cacería (no es parte del flujo normal de triage).
- Después de un incidente, para verificar si la misma TTP se usó antes sin
  ser detectada.
- Tras threat intel nueva (ej. una campaña reportada que usa una técnica
  específica).

## Flujo de trabajo
1. **Formula la hipótesis** en formato: "Si un atacante usó <TTP de MITRE
   ATT&CK>, debería dejar evidencia en <fuente de log/campo>."
   - No empieces a buscar sin esta hipótesis escrita primero.
2. **Identifica la fuente de datos**: ¿Wazuh tiene el log necesario? ¿Hace
   falta un decoder/regla nueva para verlo?
3. **Diseña la query** (Wazuh query / KQL / grep sobre logs) que probaría o
   refutaría la hipótesis.
4. **Ejecuta y documenta resultados**, incluso si son negativos — un hunt sin
   hallazgos sigue siendo información valiosa (reduce la probabilidad de esa
   TTP en el entorno).
5. Si hay hallazgo positivo, escala a `analisis-log-incidente` con la
   evidencia ya reunida.
6. Si el hunt revela una brecha de visibilidad (el log que necesitabas no
   existe o no se retiene), regístralo como gap de detección — es un
   resultado tan importante como un hallazgo de compromiso.

## Formato de documentación del hunt
```
Hipótesis: <TTP MITRE ATT&CK + justificación de por qué es relevante ahora>
Fuente de datos usada: <log source / índice>
Query ejecutada: <query o método>
Resultado: <positivo / negativo / inconcluso>
Gap de visibilidad detectado: <si aplica>
Siguiente paso: <escalar / cerrar hunt / crear regla de detección nueva>
```

## Principios
- Sin hipótesis previa no es threat hunting, es "ver qué hay" — y eso no
  escala ni es repetible.
- Un resultado negativo bien documentado tiene valor; no lo descartes.
- Si el hunt revela que falta visibilidad, eso es tan accionable como
  encontrar un IOC.
