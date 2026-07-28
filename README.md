# SOC-Implementation
Implementación de un escenario de detección y respuesta.


Red física de tu casa (bridged) — ej. 192.168.1.0/24
│
├── Laptop (Linux Mint) — 192.168.1.10
│    └── Kali Linux (VMware, Bridged) — 192.168.1.11   ← atacante
│
└── Desktop (Windows 11 + VMware Workstation Pro)
     ├── Splunk Manager (Ubuntu Server, Bridged) — 192.168.1.12
     ├── Windows 10/Server víctima (Bridged) — 192.168.1.13
     └── Ubuntu Server víctima (Bridged) — 192.168.1.14
Especificación de cada VM
VM	OS	vCPU	RAM	Disco	Rol
Splunk Manager	Ubuntu Server 22.04 LTS	2	6-8 GB	60 GB	Recibe logs, corre Splunk Enterprise + ES app
Víctima Windows	Windows 10/Server 2019	2	4 GB	60 GB	Genera telemetría (con Sysmon)
Víctima Linux	Ubuntu Server 22.04	1-2	2 GB	20 GB	Genera telemetría (auditd/rsyslog)
Kali (laptop)	Kali Linux	2	4 GB	40 GB	Ataques simulados
