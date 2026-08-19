# Runbook 02 — Actividad Anómala en Contenedor (MITRE T1059 / T1552.001)

## Contexto
Detección vía Falco standalone (driver modern_ebpf), corriendo directo sobre el host Docker sin Kubernetes. Reglas custom en Wazuh (`local_rules.xml`, grupo `mini-soc`), consumiendo el `file_output` nativo de Falco.

**Reglas involucradas:**
| rule.id | Nivel | Descripción | MITRE |
| --- | --- | --- | --- |
| 100200 | 8 | Falco: shell interactivo (Terminal shell in container) | T1059 |
| 100201 | 10 | Falco: lectura de archivo sensible (Read sensitive file untrusted) | T1552.001 |

Nota sobre el tag MITRE: Falco etiqueta la lectura de `/etc/shadow` como T1555 (Credentials from Password Stores) por defecto, pero la regla custom usa T1552.001 (Credentials In Files), que es la sub-técnica más precisa para "archivo con credenciales expuesto", en vez de copiar el tag genérico.

## Govern
- Autoridad de decisión: aislar o detener un contenedor en producción requiere aprobación de Tier 2/on-call, no acción unilateral del analista.
- Si el contenedor maneja datos personales o financieros, aplica el mismo árbol de reporte regulatorio del Runbook 01.

## Identify
- Activo afectado: contenedor target-host sobre el host Docker monitoreado por Falco standalone (modo modern_ebpf, sin Kubernetes).
- Confirmar si el contenedor tiene acceso a secretos o volúmenes compartidos con otros servicios — determina el radio de impacto real.

## Protect
- Controles preventivos ya aplicados en otros proyectos del portafolio (ej. pipeline DevSecOps con imágenes fijadas por hash, ejecución sin root, --cap-drop ALL), lo que es el estándar a exigir en cualquier contenedor de este entorno, aunque target-host no lo implementa por ser el objetivo de prueba.

## Detect
- Filtro: rule.id:(100200 OR 100201) en Threat Hunting → Events.
- 100200 (nivel 8): "Terminal shell in container", regla custom, campo output_fields.proc.name, MITRE T1059.
- 100201 (nivel 10): "Read sensitive file untrusted", regla custom, campo output_fields.fd.name, MITRE T1552.001 (sub-técnica más precisa que el T1555 genérico que trae Falco por defecto).
- Ruido de fondo conocido: wazuh-modulesd (el propio manager) dispara 100201 periódicamente al leer /etc/shadow como parte de su chequeo interno, por lo que se debe descartar antes de escalar.

## Respond
- Confirmar usuario, proceso y comando exacto que disparó la alerta.
- Escalar el aislamiento del contenedor a Tier 2/on-call.
- Investigar el vector de acceso inicial: ¿cómo se llegó a ejecutar un shell ahí?

## Recover
- Reconstruir el contenedor desde una imagen limpia, nunca se debe reutilizar el contenedor comprometido.
- Retroalimentar a Identify: si el vector de acceso revela una debilidad de diseño (ej. imagen sin hardening), actualizar el inventario de riesgos de esa imagen base. Retroalimentar a Govern: evaluar si se requiere una política de hardening obligatoria (non-root, cap-drop) antes de producción.

## Contexto Bancario
Mismo criterio de relojes regulatorios que el Runbook 01 (CMF 30 min / ANCI 3h / Ley 21.719 si aplica), teniendo en cuenta que la naturaleza técnica del hallazgo (runtime vs. red) no cambia la obligación de reporte, solo el análisis técnico previo.