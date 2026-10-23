---
name: generar-reporte-incidente
description: Genera el reporte post-incidente alineado al ciclo de vida de NIST SP 800-61 (preparación, detección/análisis, contención/erradicación/recuperación, lecciones aprendidas). Úsala al cerrar un incidente P1/P2 que ya pasó por analisis-log-incidente y deteccion-persistencia.
---

# Generar Reporte de Incidente

## Objetivo
Producir un reporte post-incidente completo y defendible, útil tanto para
cerrar el caso como para prevenir recurrencia — no un resumen informal.

## Cuándo se activa
- Al cerrar formalmente un incidente clasificado P1/P2.
- Nunca antes de haber corrido `deteccion-persistencia` en los activos
  afectados — el reporte debe incluir ese resultado.

## Estructura del reporte (NIST SP 800-61)

1. **Resumen ejecutivo**: qué pasó, impacto, estado actual, en 3-5 líneas,
   sin jerga técnica.
2. **Línea de tiempo**: eventos en orden cronológico con timestamps,
   distinguiendo evidencia directa de hipótesis que no se pudo confirmar.
3. **Detección y análisis**: cómo se detectó, qué IOCs se identificaron,
   resultado del enriquecimiento (enriquecimiento-ioc).
4. **Contención, erradicación y recuperación**: acciones tomadas, en qué
   orden, y evidencia de que funcionaron.
5. **Resultado de verificación de persistencia**: resultado explícito del
   checklist de `deteccion-persistencia` (positivo/negativo, qué se revisó).
6. **Causa raíz**: por qué ocurrió, no solo qué ocurrió.
7. **Lecciones aprendidas**: qué falló en la detección/prevención, y
   recomendaciones de `hardening-recomendaciones` si aplica.
8. **Clasificación final**: confirmado / falso positivo / inconcluso, con
   justificación.

## Principios
- Toda afirmación de "se contuvo" o "no hay persistencia" debe tener
  evidencia citada, no ser una suposición.
- Si algo quedó sin resolver o sin verificar, dilo explícitamente en el
  reporte — no lo omitas para que se vea más limpio.
- El reporte debe ser entendible por alguien que no vivió el incidente en
  tiempo real.
- Nunca cierres un incidente como falso positivo sin justificación
  explícita (regla ya establecida para el agente SOC).
