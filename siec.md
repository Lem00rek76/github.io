---
layout: page
title: "Sieć"
permalink: /siec
---

## Podstawowa topologia

- **Sieć domowa oparta na MikroTik-ach**
- **Główna podsieć LAN:** `10.10.20.0/24`
- **Adres rpi5:** `10.10.20.10`  
  (IP, pod którym działają m.in. Pi-hole, Traefik i usługi za Traefikiem)

## Router MikroTik

- Router MikroTik pełni rolę głównego routera i serwera DHCP.
- DHCP rozdaje adresy klientom w sieci `10.10.20.0/24`.

### Ustawienia DNS w DHCP

- **DNS Server** dla klientów ustawiony na adres Pi-hole:
  - `10.10.20.10`
- Dzięki temu:
  - cały ruch DNS z klientów (Windows/Linux) przechodzi przez Pi-hole,
  - blokowanie reklam i lokalne rekordy DNS działają centralnie.

## Lokalne domeny *.lab

W Pi-hole zdefiniowane są lokalne rekordy typu **A** dla domen w strefie `*.lab`, wskazujące na rpi5 (`10.10.20.10`), np.:

- `portainer.lab` → `10.10.20.10`
- `gitea.lab` → `10.10.20.10`
- (docelowo także: `grafana.lab`, `kuma.lab`, `loki.lab`, `traefik.lab`, itp.)

Dzięki temu:

- z dowolnej maszyny w LAN można używać przyjaznych nazw zamiast adresów IP,
- Traefik może routować ruch na podstawie hostów (np. `Host("portainer.lab")`).

## Testy rozwiązywania DNS

Przykładowe komendy testowe z klientów:

```bash
nslookup portainer.lab
nslookup gitea.lab
Oczekiwany wynik: odpowiedź DNS wskazująca na 10.10.20.10.
