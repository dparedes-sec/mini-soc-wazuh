# Runbook 01 — Fuerza Bruta SSH (MITRE T1110 / T1110.001)


## Contexto
Detección basada en el ruleset nativo de Wazuh (0095-sshd_rules.xml y
0090-pam_rules.xml), sin reglas custom. Correlación confirmada en el labsobre el agente `target-host`.

**Reglas involucradas:**
| rule.id | Nivel | Descripción |
| --- | --- | --- |
| 5760 | 5 | sshd: authentication failed (intento individual) |
| 5503 | 5 | PAM: User login failed (mismo evento, vía PAM) |
| 5763 | 10 | sshd: brute force trying to get access to the system |
| 5551 | 10 | PAM: Multiple failed logins in a small period of time |

Nota: 5763 y 5551 son dos correlaciones independientes (sshd y PAM) sobre la misma actividad, por lo que confirmar ambas no es redundante, es la señal de que dos capas del sistema coinciden en el diagnóstico.

## Govern
- Autoridad de decisión: un analista SOC junior detecta y documenta, pero el bloqueo de la IP origen lo aprueba y ejecuta Tier 2/on-call, salvo que exista un runbook de emergencia pre-aprobado para este patrón específico.
- Política de referencia: rotación y complejidad de contraseñas para cuentas con acceso SSH; en este lab, labuser con password débil es intencional para efectos de prueba.
- Si el host afectado maneja datos financieros o personales, este es el punto donde se activa la obligación de evaluar el reporte regulatorio: CMF (30 min, RAN Cap. 20-8), ANCI (3h, Ley 21.663 si el banco es OIV), Ley 21.719 (datos personales, "sin dilación indebida"). El analista SOC no decide el reporte, pero su timestamp de detección arranca esos relojes.

## Identify
- Activo afectado: target-host (agente Wazuh registrado). En un entorno real, se consulta el registro de riesgos del activo (criticidad, datos que almacena) antes de decidir la urgencia de la contención.

## Protect
-Control preventivo recomendado: autenticación por llave SSH en vez de password. No estaba implementado en el lab a propósito, para poder generar la señal de ataque.

## Detect
- Filtro: rule.id:(5760 OR 5763 OR 5551) en Threat Hunting → Events.
- 5760 (nivel 5, sshd) y 5503 (nivel 5, PAM): intentos individuales fallidos.
- 5763 (nivel 10, sshd) y 5551 (nivel 10, PAM): correlación de fuerza bruta, disparada en paralelo por dos subsistemas distintos, que fueron MITRE T1110 (5763) y T1110.001 (5760, Password Guessing).
- Ruleset nativo de Wazuh (0095-sshd_rules.xml / 0090-pam_rules.xml), sin reglas custom.

## Respond
- Confirmar si algún intento tuvo éxito (login posterior desde el mismo srcip).
- Si hubo compromiso: invalidar sesión y credenciales del usuario objetivo.
- Escalar el bloqueo de la IP origen a Tier 2/on-call.
- Comunicar el hallazgo según el árbol de escalamiento definido en Govern.

## Recover
- Restaurar acceso SSH normal una vez contenida la amenaza.
- Reforzar con autenticación por llave.
- Retroalimentar a Identify: actualizar el registro de riesgos del host si se detectó un patrón repetible. Retroalimentar a Govern: evaluar si la política de contraseñas necesita ajuste.

## Contexto Bancario (ejercicio de proceso, no integración real)
En un banco chileno regulado por la CMF, este tipo de incidente dispara relojes que este runbook por sí solo no resuelve:
- Reporte a la CMF vía plataforma digital en máximo 30 minutos (RAN Cap. 20-8).
- Si el banco es Operador de Importancia Vital (Ley 21.663), reporte a la ANCI en 3 horas.
- Si hubo acceso exitoso a datos personales, notificación a la Agencia de Protección de Datos "sin dilación indebida" (Ley 21.719).

El analista SOC no decide si se reporta, pero su timestamp de detección
es el dato que arranca esos relojes.