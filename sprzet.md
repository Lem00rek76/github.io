---
layout: page
title: "Sprzęt"
permalink: /sprzet
---

## Główny serwer

- **Model:** Raspberry Pi 5 8GB + 256GB NVme (rpi5)  
- **Rola:** główny serwer homelabu 
- **System operacyjny:** Ubuntu 25.10 „Questing Quokka”

## Serwer Proxmox
- **Model:** Intel i7-8550U 32GB RAM + 1TB NVME (proxmox)
- **Rola:** host maszyn wirtualnych
- **System Operacyjny:** Proxmox VE 9.1

## Oprogramowanie bazowe

- **Docker** – zainstalowany ręcznie z oficjalnego repozytorium Dockera (nie z repo starej wersji Ubuntu)
- **Docker Compose** – do zarządzania stackami usług

## Struktura katalogów dla usług Docker

Wszystkie usługi Dockera trzymane są w katalogu `/srv/docker/`:

- `/srv/docker/traefik/` – reverse proxy Traefik
- `/srv/docker/portainer/` – Portainer (GUI do zarządzania Dockerem)
- `/srv/docker/gitea/` – serwer Gitea + Postgres
- `/srv/docker/monitoring/` – narzędzia monitoringu (Grafana, Prometheus, Loki, Uptime Kuma, itp.)
- `/srv/docker/pihole/` – Pi-hole (DNS + blokowanie reklam)
- (w przyszłości kolejne serwisy - m.in. Vaultwarden)

Dzięki temu wszystkie usługi Dockera są w jednym, logicznym miejscu (`/srv/docker/`), łatwym do backupu i przeglądu.
