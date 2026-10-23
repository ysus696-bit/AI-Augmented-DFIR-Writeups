# Metodología: validación humano-en-el-bucle sobre agentes de IA en DFIR

Este documento describe el marco general que aplico en todos los casos de este repositorio. Cada writeup individual hace referencia a esto en vez de repetirlo.

## Principios

1. **El agente propone, el analista confirma.** Ningún IOC, timestamp o conclusión del agente se considera válido hasta ser contrastado contra la evidencia cruda.
2. **Las decisiones de contexto e intención son humanas.** El agente puede identificar patrones, pero la interpretación de impacto de negocio y severidad las hago yo.
3. **Todo ajuste de skill/prompt queda documentado.** Si un caso revela una debilidad del agente (alucinación, sesgo, omisión), el cambio correspondiente se versiona con su changelog.
4. **La trazabilidad es el entregable, no solo la respuesta final.** El valor del proceso está en poder mostrar exactamente qué hizo el agente, qué corregí y por qué.

## Flujo estándar

1. Ingesta de evidencia → agente hace parseo y normalización inicial
2. Agente propone hallazgos preliminares (IOCs, timeline, patrones)
3. Analista valida cada hallazgo contra la fuente original
4. Analista descarta falsos positivos / corrige errores del agente
5. Agente asiste en mapeo MITRE ATT&CK → analista confirma
6. Analista redacta interpretación de negocio, severidad y recomendaciones
7. Si se detectó una debilidad recurrente del agente → se actualiza la skill y se documenta en su changelog

## Qué NUNCA delego al agente

- Confirmación final de un IOC sin verificación cruzada
- Juicio sobre intención del atacante o severidad real del incidente
- Decisiones de contención/remediación
- Atribución

## Señales de alerta que vigilo en el output del agente

- Timestamps sin fuente explícita citada
- IOCs sin referencia al log/artefacto exacto de origen
- Conclusiones que combinan varias fuentes sin mostrar el razonamiento intermedio
- Lenguaje de alta confianza sobre datos ambiguos o incompletos
