# AI-Augmented DFIR Writeups

> Investigaciones de ciberseguridad (DFIR / Blue Team) documentadas con un enfoque de **validación humano-en-el-bucle sobre agentes de IA**: cómo delego tareas analíticas a agentes, cómo valido y corrijo sus resultados, y cómo itero sobre los prompts/skills para mejorar su fiabilidad en contextos forenses reales.

## Sobre este repositorio

Este repositorio no es solo una colección de writeups de retos (HTB Sherlocks, Holmes CTF, CyberDefenders, etc.). Cada documento está estructurado para mostrar **dos capas de trabajo en paralelo**:

1. **El caso técnico en sí** — el análisis forense/investigativo completo.
2. **El proceso de trabajo con agentes de IA** — qué tareas delegué, qué valida un analista humano, dónde el agente se equivocó o alucinó, y cómo ajusté mis skills/prompts a partir de eso.

El objetivo es demostrar una competencia que cada vez más equipos SOC necesitan: **operar agentes de IA con criterio y gobernanza**, no solo saber usarlos.

## Por qué este enfoque

- Cualquiera puede pegar un prompt y copiar una respuesta. Lo que diferencia a un analista es **saber cuándo desconfiar del output del agente** y tener un proceso repetible de validación.
- Los agentes de IA en contextos forenses pueden alucinar timestamps, IOCs o relaciones causales — documentar cómo detecto y corrijo eso es evidencia directa de pensamiento crítico técnico.
- Esta es la dirección hacia la que se mueve la industria (*AI-augmented triage*, *human-in-the-loop validation*, *agent governance*).

## Estructura del repositorio

```
ai-augmented-dfir/
├── README.md                          ← este archivo
├── TEMPLATE-writeup.md                ← plantilla base para cada nuevo caso
├── skills/                            ← versiones de las skills/prompts usadas
│   ├── deteccion-persistencia/
│   │   ├── v1-SKILL.md
│   └── generar-reporte-incidente/
│       ├── v1-SKILL.md
│   └── validacion-remediacion/
│       ├── v1-SKILL.md
│   └── threat-hunting-proactivo/
│       ├── v1-SKILL.md
├── casos/
│   ├── 2026-XX-watchmans-residue/
│   │   ├── writeup.md
│   └── 2026-XX-<siguiente-caso>/
│       └── ...
└──metodologia.md                 ← marco general de validación que uso en todos los casos
```

## Marco de validación (resumen)

| Etapa | Quién la ejecuta | Quién valida |
|---|---|---|
| Parseo inicial de evidencia (logs, PCAP, JSON) | Agente de IA | — |
| Identificación de patrones / posibles IOCs | Agente de IA | Analista (yo) |
| Confirmación de IOCs contra evidencia cruda | Analista (yo) | — |
| Construcción de línea de tiempo | Agente de IA (borrador) | Analista (yo, ajuste final) |
| Interpretación de contexto de negocio / intención | Analista (yo) | — |
| Mapeo a MITRE ATT&CK | Agente de IA (propuesta) | Analista (yo, confirmación) |
| Recomendaciones de mitigación | Analista (yo) | — |

> Detalle completo del marco en [`metodologia.md`](metodologia.md).

## Stack y herramientas

- **Agentes/orquestación:** OpenCode con skills propias
- **Análisis forense:** Wireshark, jq, grep/awk, CyberChef
- **Framework de referencia:** MITRE ATT&CK, NIST SP 800-61 (incident handling)
