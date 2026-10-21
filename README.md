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
- Esta es la dirección hacia la que se mueve la industria (*AI-augmented triage*, *human-in-the-loop validation*, *agent governance*) y quiero que mi portafolio lo refleje con casos reales, no solo teoría.

## Estructura del repositorio

```
ai-augmented-dfir/
├── README.md                          ← este archivo
├── TEMPLATE-writeup.md                ← plantilla base para cada nuevo caso
├── skills/                            ← versiones de las skills/prompts usadas
│   ├── analisis-log-incidente/
│   │   ├── v1-SKILL.md
│   │   ├── v2-SKILL.md
│   │   └── CHANGELOG.md
│   └── triage-alertas-siem/
│       ├── v1-SKILL.md
│       └── CHANGELOG.md
├── casos/
│   ├── 2026-XX-watchmans-residue/
│   │   ├── writeup.md
│   │   ├── evidencia/                 ← capturas, extractos de logs (sanitizados)
│   │   └── metricas.md
│   └── 2026-XX-<siguiente-caso>/
│       └── ...
└── docs/
    └── metodologia.md                 ← marco general de validación que uso en todos los casos
```

### Por qué esta estructura

- **`skills/`** — guardo el historial de versiones de cada skill/prompt que uso, con su changelog. Esto es la evidencia más fuerte de iteración disciplinada: un reclutador técnico puede ver exactamente qué cambié y por qué.
- **`casos/`** — cada caso es una carpeta independiente con su writeup, evidencia de respaldo y métricas del proceso. Mantiene todo trazable y fácil de navegar.
- **`docs/metodologia.md`** — documento único donde explico mi marco de trabajo general (no se repite en cada writeup), para que cada caso pueda enfocarse en lo específico.

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

> Detalle completo del marco en [`docs/metodologia.md`](docs/metodologia.md).

## Índice de casos

| Caso | Fuente | Categoría | Tema principal | Writeup |
|---|---|---|---|---|
| The Watchman's Residue | HTB Holmes CTF 2025 | DFIR / AI Abuse | Manipulación de chatbot IA para filtración de credenciales RMM | [`casos/2026-XX-watchmans-residue/`](casos/2026-XX-watchmans-residue/) |

## Stack y herramientas

- **Agentes/orquestación:** OpenCode con skills propias (`analisis-log-incidente`, `triage-alertas-siem`)
- **Análisis forense:** Wireshark, jq, grep/awk, CyberChef
- **Framework de referencia:** MITRE ATT&CK, NIST SP 800-61 (incident handling)

## Sobre mí

José Andrés Rojas Sojo ("yisus") — Técnico de soporte N2 con 6+ años en infraestructura de redes ISP, actualmente en transición hacia roles de SOC Analyst / Pentesting. Certificación eJPT, cursando Ingeniería en Ciberseguridad.

- Writeups ofensivos: [htb-writeups](https://github.com/yisus666-bit/htb-writeups)
- Herramientas de recon: [ReconSuite](https://github.com/ysus696-bit/ReconSuite)
