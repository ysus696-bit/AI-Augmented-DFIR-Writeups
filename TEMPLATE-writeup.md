# [Nombre del caso] — [Fuente: HTB Holmes CTF 2025 / CyberDefenders / etc.]

**Fecha:** [YYYY-MM-DD]
**Categoría:** [DFIR / SOC / Threat Hunting / Malware Analysis]
**Dificultad:** [Easy / Medium / Hard]
**Agente/skill utilizada:** [nombre y versión, ej. `analisis-log-incidente v2`]

---

## 1. Resumen ejecutivo

> 2-3 líneas: qué pasó en el caso, cómo se descubrió, y el hallazgo principal. Escrito como si fuera para un stakeholder no técnico.

[Tu resumen aquí]

---

## 2. Contexto del escenario

[Descripción del caso tal como la plantea el reto: organización ficticia, qué se sospecha, qué artefactos se entregan]

**Artefactos proporcionados:**
- [archivo 1 — tipo, tamaño]
- [archivo 2 — tipo, tamaño]

---

## 3. Metodología: humano + agente de IA

Esta sección es el núcleo diferenciador del documento. Sé específico y honesto — incluye los errores del agente, no solo los aciertos.

### 3.1 División de tareas

| Tarea | Ejecutada por | Validada por |
|---|---|---|
| [ej. Parseo inicial del PCAP] | Agente | — |
| [ej. Identificación de endpoint de la API] | Agente | Yo |
| [ej. ...] | ... | ... |

### 3.2 Prompt / instrucción inicial dada al agente

```
[pega aquí el prompt o la skill que usaste tal cual, o un resumen fiel]
```

### 3.3 Qué entregó el agente (primer borrador)

[Resumen o extracto del output inicial del agente — sin editar]

### 3.4 Validación y corrección

> Aquí va lo más valioso del documento: ejemplos concretos de revisión crítica.

- **Hallazgo del agente:** [ej. "marcó la IP X como IOC"]
  **Mi validación:** [ej. "revisé el PCAP crudo y confirmé que correspondía a tráfico interno legítimo del mismo segmento — descartado"]

- **Hallazgo del agente:** [ej. "propuso un timestamp para el primer acceso"]
  **Mi validación:** [ej. "el timestamp no coincidía con la zona horaria del log original — corregido manualmente"]

- [agrega tantos pares hallazgo/validación como hayan ocurrido en el caso real]

### 3.5 Ajuste de la skill/prompt (si aplica)

[Si tuviste que modificar tu skill.md o prompt a partir de este caso, documenta el cambio]

```diff
- [línea anterior de la skill]
+ [línea nueva, y por qué]
```

> Changelog completo en [`skills/<nombre-skill>/CHANGELOG.md`](../../skills/)

---

## 4. Análisis técnico detallado

> Aquí va el writeup "tradicional", ya validado — el análisis forense completo paso a paso.

### 4.1 [Primer hallazgo / pregunta del reto]

[Desarrollo técnico]

### 4.2 [Segundo hallazgo / pregunta del reto]

[Desarrollo técnico]

### 4.3 Línea de tiempo del incidente

| Timestamp | Evento | Fuente de evidencia |
|---|---|---|
| [fecha/hora] | [qué pasó] | [archivo/log de origen] |

---

## 5. Mapeo a MITRE ATT&CK

| Técnica | ID | Táctica | Evidencia |
|---|---|---|---|
| [nombre técnica] | [T####] | [táctica] | [referencia a la sección 4] |

---

## 6. Indicadores de compromiso (IOCs)

| Tipo | Valor | Confianza | Validado por |
|---|---|---|---|
| IP | [valor] | [Alta/Media/Baja] | [Agente / Yo / Ambos] |
| Hash | [valor] | | |
| Dominio | [valor] | | |

---

## 7. Métricas del proceso

| Métrica | Valor |
|---|---|
| Tiempo total de investigación | [horas] |
| % de hallazgos iniciales del agente que requirieron corrección | [%] |
| Falsos positivos propuestos por el agente | [cantidad] |
| Tiempo estimado ahorrado vs. análisis 100% manual | [estimación] |

---

## 8. Lecciones aprendidas

### Dónde el agente aportó más valor
- [punto 1]
- [punto 2]

### Dónde requirió más supervisión
- [punto 1]
- [punto 2]

### Para operacionalizar esto en un SOC real
- [ej. cuándo escalar automáticamente a un humano]
- [ej. qué tipo de decisiones nunca deberían delegarse al agente]
- [ej. riesgos de alucinación específicos de este tipo de evidencia]

---

## 9. Recomendaciones de mitigación (para la organización del caso)

- [recomendación 1]
- [recomendación 2]

---

## Referencias

- [Enlace al reto original, si es público]
- [Documentación de herramientas usadas]
