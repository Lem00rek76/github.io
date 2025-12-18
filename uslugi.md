---
layout: page
title: "Usługi"
permalink: /uslugi
---

# Usługi

## Przegląd

Na rpi5 z Dockerem działają (lub są planowane) następujące kluczowe usługi:

- **Pi-hole** – DNS + blokowanie reklam
- **Traefik** – reverse proxy dla usług HTTP/HTTPS
- **Portainer** – webowe GUI do zarządzania Dockerem
- **Gitea** – serwer Git z webowym interfejsem
- **Monitoring / logi** – Grafana, Prometheus, Loki, Uptime Kuma (w trakcie)

---

## Pi-hole (DNS + blokowanie reklam)

- Uruchomiony w Dockerze na rpi5.
- Nasłuchuje na:
  - portach DNS `53/udp` i `53/tcp` (zmapowane na `10.10.20.10`),
  - panel WWW na porcie `80` (lub innym, jeśli wymaga tego konfiguracja z Traefikiem).
- MikroTik jest skonfigurowany tak, aby:
  - używać Pi-hole jako głównego DNS (`10.10.20.10`),
  - przekazywać ten DNS klientom przez DHCP.

W Pi-hole zdefiniowane są lokalne rekordy:

- `portainer.lab` → `10.10.20.10`
- `gitea.lab` → `10.10.20.10`
- oraz kolejne wpisy dla innych usług.

---

## Traefik (reverse proxy)

Traefik działa jako centralny reverse proxy dla usług HTTP/HTTPS w sieci `proxy`.

**Compose (w skrócie):**

- image: `traefik:v2.11`
- porty:
  - `80:80` – HTTP (entrypoint `web`)
  - `443:443` – HTTPS (entrypoint `websecure`, w przyszłości z certyfikatami)
  - `8085:8080` – dashboard Traefika (niezabezpieczony, tylko w LAN)

**Konfiguracja głównych entrypointów i providera Docker:**

- `--entrypoints.web.address=:80`
- `--entrypoints.websecure.address=:443`
- `--providers.docker=true`
- `--providers.docker.exposedbydefault=false`

**Dashboard Traefika:**

- URL: `http://10.10.20.10:8085`
- Włączony parametrami:
  - `--api.dashboard=true`
  - `--api.insecure=true`

Traefik oraz usługi za nim działają w współdzielonej sieci Docker `proxy` (`external: true`).

---

## Portainer

Portainer służy do zarządzania Dockerem przez GUI.

- Działa jako kontener w sieci `proxy`.
- Ma podpięty:
  - `docker.sock` (do zarządzania Dockerem),
  - katalog `./data` na swoje dane.

**Integracja z Traefikiem:**

- `traefik.enable=true`
- `traefik.http.routers.portainer.rule=Host("portainer.lab")`
- `traefik.http.routers.portainer.entrypoints=web`
- `traefik.http.services.portainer.loadbalancer.server.port=9000`

**Efekt:**

- Portainer jest dostępny pod adresem:  
  `http://portainer.lab`

---

## Gitea (Git serwer + GUI)

Gitea działa w Dockerze, korzystając z bazy Postgres.

### Skład stacka

- **Postgres (db)** – `postgres:16`
  - `POSTGRES_DB: gitea`
  - `POSTGRES_USER: gitea`
  - `POSTGRES_PASSWORD: "Gitea_2024!_db"` (w YAML zapisane w cudzysłowach)
  - dane w volume: `./postgres:/var/lib/postgresql/data`

- **Gitea (aplikacja)** – `gitea/gitea:1.22-rootless`
  - działa jako rootless:
    - `USER_UID: 1000` (użytkownik `marcin`)
    - `USER_GID: 1003` (grupa użytkownika)
  - konfiguracja bazy przez zmienne:
    - `GITEA__database__DB_TYPE: postgres`
    - `GITEA__database__HOST: db:5432`
    - `GITEA__database__NAME: gitea`
    - `GITEA__database__USER: gitea`
    - `GITEA__database__PASSWD: "Gitea_2024!_db"`
  - dane aplikacji: `./gitea:/var/lib/gitea`
  - porty:
    - `3002:3000` – HTTP (tymczasowy dostęp: `http://10.10.20.10:3002`)
    - `2222:22` – SSH do repo (opcjonalny)

### Uprawnienia i instalacja

- Katalogi `./gitea` i `./postgres` mają nadane prawa:
  - `chown -R 1000:1003 gitea postgres`
- Instalacja Gitea przeszła pomyślnie:
  - logowanie działa,
  - parametry bazy zgodne ze zmiennymi środowiskowymi.

### Parametry serwera Gitea (podczas instalacji)

- **Database:**
  - Type: PostgreSQL
  - Host: `db:5432`
  - User: `gitea`
  - Password: `Gitea_2024!_db`
  - DB name: `gitea`

- **Server:**
  - HTTP Port: `3000` (wewnątrz kontenera)
  - Application URL: `http://10.10.20.10:3002/` (na czas konfiguracji)
  - Server Domain: `10.10.20.10`
  - SSH Port: `2222` (jeśli używany)

### Plan docelowy

- Dodać Gitea do sieci `proxy`.
- Wystawić Gitea przez Traefika:
  - `http://gitea.lab` → Gitea (port 3000 w kontenerze)
- Zmienić `ROOT_URL` / `Application URL` w Gitea na:
  - `http://gitea.lab/`

---

## Monitoring i logi (plan)

W trakcie konfiguracji / planowania:

- **Grafana**
- **Prometheus**
- **Loki + Promtail**
- **Uptime Kuma**

Lokalizacja stacka:

- katalog: `/srv/docker/monitoring/`

Docelowo:

- dostępne przez Traefika:
  - `http://grafana.lab`
  - `http://kuma.lab`
  - `http://loki.lab`
- Loki wymagał poprawienia uprawnień do katalogów, aby mógł pisać do volume.
