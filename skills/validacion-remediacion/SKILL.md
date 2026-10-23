---
name: validacion-remediacion
description: Estructura el retest de hallazgos ya reportados para confirmar si una remediación aplicada realmente cerró la vulnerabilidad, sin volver a correr todo el engagement desde cero. Úsala cuando el cliente pide validar un fix sobre un hallazgo de generar-reporte-pentest previo.
---

# Validación de Remediación (Retest)

## Objetivo
Confirmar, de forma puntual y eficiente, si un hallazgo previamente
reportado fue corregido — sin repetir todo el alcance del pentest original.

## Entradas esperadas
- Reporte original del hallazgo (de `generar-reporte-pentest` o
  `enumeracion-vulnerabilidades`).
- Confirmación de qué cambio se aplicó (si el cliente lo compartió).

## Flujo de trabajo
1. Recupera el hallazgo original: vector, evidencia, método usado para
   confirmarlo la primera vez.
2. Repite **el mismo método** usado originalmente — un retest con un método
   distinto no es comparable y puede dar falsos negativos.
3. Clasifica el resultado:
   - **Remediado**: el vector ya no es explotable con el mismo método.
   - **Remediado parcialmente**: el vector original se cerró pero persiste
     una variante (documéntala como hallazgo nuevo, no como el mismo).
   - **No remediado**: el vector sigue funcionando igual que antes.
   - **Remediación no verificable**: no se pudo confirmar por limitaciones
     de acceso/alcance — dilo explícitamente, no asumas que está resuelto.
4. Si el hallazgo persiste, verifica si el cambio reportado por el cliente
   realmente se aplicó en el activo (a veces el fix se aplicó en el ambiente
   equivocado).

## Formato de salida
```
Hallazgo original: <referencia al reporte>
Método de verificación: <mismo método que el hallazgo original>
Resultado: <Remediado / Parcial / No remediado / No verificable>
Evidencia: <...>
Hallazgo nuevo (si aplica): <variante no cubierta por el fix original>
```

## Principios
- Un retest usa el mismo método que el hallazgo original — cambiar el método
  invalida la comparación.
- "No verificable" es una respuesta válida y honesta cuando el alcance o
  acceso no permite confirmar — no lo fuerces a "remediado" ni a "no
  remediado" sin evidencia real.
