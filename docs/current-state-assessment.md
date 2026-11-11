\## Compose assessment



No Docker Compose definition was present in the repository at the time

of assessment.



The repository contained a standalone Dockerfile for the SSH target,

but did not contain a reproducible orchestration definition for Wazuh,

the target host, networking, volumes, certificates, or service

dependencies.



Consequences:



\- `docker compose config` could not be executed.

\- The deployment could not be reconstructed from the repository alone.

\- Network exposure and persistent storage were not defined as code.

\- Exact Wazuh component versions were not pinned within the repository.

\- The relationship between Wazuh and the target host depended on

&#x20; undocumented external deployment state.



\### Finding REP-001



\- Severity: Medium

\- Confidence: High

\- Type: Reproducibility and configuration management

\- Incident status: Not an incident



\### Required action



Create a versioned Compose definition after selecting and validating

compatible Wazuh component versions. The deployment must use a

project-specific network and named volumes and must avoid interfering

with unrelated Docker Desktop projects.



\## Docker Desktop inventory



At assessment time, Docker Desktop was operational in Linux container

mode.



No Wazuh, Falco, or `target-host` containers or images were present.

Existing containers, networks, images, and anonymous volumes belonged

to unrelated local projects and were excluded from the Mini SOC scope.



The project must not use broad cleanup commands such as:



\- `docker system prune`

\- `docker volume prune`

\- deletion of unrelated networks

\- deletion of unrelated containers



All Mini SOC resources must use explicit names or Compose project

labels so that cleanup remains scoped to this laboratory.

