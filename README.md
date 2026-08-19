# Mini-SOC — Wazuh desde cero

SIEM autocontenido en Docker (WSL2/Ubuntu), sin depender de infraestructura
externa. Todo corre en un solo host, con 3 fuentes de detección propias.

## Arquitectura
- **Wazuh single-node** (manager + indexer + dashboard), levantado desde el
  repo oficial `wazuh-docker`.
- **target-host**: contenedor Ubuntu con SSH, usuario de prueba y rsyslog,
  usado como endpoint monitoreado para fuerza bruta SSH.
- **Falco standalone**: corriendo directo sobre el host Docker (sin
  Kubernetes), driver `modern_ebpf`, output nativo a archivo — sin
  Falcosidekick.

## Detecciones implementadas
| Fuente | Regla(s) | MITRE ATT&CK |
|---|---|---|
| SSH Brute Force (nativo) | 5760, 5763, 5551 | T1110, T1110.001 |
| Falco — shell interactivo | 100200 (custom) | T1059 |
| Falco — lectura de archivo sensible | 100201 (custom) | T1552.001 |
| Vulnerability Detection | CVE-2022-41409 (libpcre2-8-0) | — |

## Runbooks
Los 3 runbooks siguen **NIST SP 800-61 Rev. 3**, estructurados sobre las
6 funciones del NIST CSF 2.0 (Govern, Identify, Protect, Detect, Respond,
Recover) — no el modelo lineal de 4 fases de la Rev. 2, que NIST retiró
oficialmente en abril de 2025. Cada runbook incluye además una sección de
"Contexto Bancario": un ejercicio de proceso sobre los plazos regulatorios
que enfrentaría un banco chileno real (CMF, ANCI, Ley 21.719).
**Esto es documentación de proceso, no una integración real con esas
entidades.**

## Estructura
```text
mini-soc-wazuh/
├── config/
│   ├── agent-additions.xml
│   └── manager-additions.xml
├── dashboards/
│   └── mini-soc-dashboard.ndjson
├── detection/
│   └── wazuh/
│       └── local_rules.xml
├── evidence/
│   ├── CVE-2022-41409.md
│   └── vulnerability-detection-inventory.png
├── falco/
│   └── falco.yaml
├── runbooks/
│   ├── RUNBOOK-01-ssh-bruteforce.md
│   ├── RUNBOOK-02-falco-container.md
│   └── RUNBOOK-03-vulnerability.md
├── target-host/
│   └── Dockerfile
└── README.md
```

## Notas técnicas relevantes
- Falco en WSL2 requiere forzar `engine.kind: modern_ebpf` explícitamente
  (el driver clásico no tiene módulo prebuilt para el kernel de WSL2).
- Un contenedor Ubuntu mínimo no genera `/var/log/auth.log` sin `rsyslog`
  instalado y corriendo — sshd por sí solo no escribe a archivo.
- Los archivos editados con `docker cp` deben recuperar su `chown`
  original (`root:wazuh`) antes de reiniciar el servicio correspondiente.
- Falco etiqueta algunas alertas con tags MITRE genéricos (ej. T1555 para
  lectura de `/etc/shadow`); las reglas custom de Wazuh usan la
  sub-técnica más precisa (T1552.001) en vez de copiar el tag por defecto.
