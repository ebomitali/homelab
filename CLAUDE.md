# CLAUDE.md — regole del homelab

Queste regole valgono per ogni modifica al repository.

## Struttura

- Ogni servizio è uno stack in `stacks/<nome>/docker-compose.yml`.
- I dati di uno stack stanno in una cartella con il suo nome, sotto `data/<nome>/`.
  I dati non vanno mai committati su git (la cartella `data/` è in `.gitignore`).

## Rete ed esposizione

- Tutti i servizi stanno sulla rete Docker esterna `homelab`:

  ```yaml
  networks:
    homelab:
      external: true
  ```

- I servizi non pubblicano porte sull'host. Si espongono solo tramite Traefik,
  con label su `<nome>.${DOMAIN}`, dove `DOMAIN` è la variabile nel `.env` dello stack
  (es. `DOMAIN=home.zebco.it`, quindi `portainer.home.zebco.it`).
  Le label usano `${DOMAIN:?impostare DOMAIN}`, così lo stack non parte senza dominio.
- Traefik è l'unico stack che pubblica porte (80 e 443) sull'host. La 80 reindirizza a HTTPS (443).
- Tutti i servizi web usano `entrypoints=websecure` e `tls=true` (vedi sezione TLS).
- Eccezione: AdGuard (`stacks/adguard`) pubblica DNS 53/udp e 53/tcp solo
  sull'IP LAN del server (`192.168.1.11`), così i dispositivi della rete domestica
  possono usarlo come DNS. Nessuna altra porta, e nessun bind su `0.0.0.0`.

## TLS

- I certificati sono creati con `mkcert` (CA locale) e stanno in `data/traefik/certs/`.
  Non vanno mai committati. La chiave della CA (`rootCA-key.pem`) resta solo sul server.
- La configurazione TLS per Traefik sta in `data/traefik/dynamic/tls.yml`, copia di
  `stacks/traefik/tls.yml.example`.
- Ogni dispositivo che usa l'homelab deve avere installato `rootCA.pem` (solo il certificato, mai la chiave).

## Immagini

- Ogni immagine ha una versione fissa (es. `traefik:v3.1.6`).
- Mai usare `latest` né tag senza versione.

## Segreti

- I segreti stanno solo nel file `.env` (dentro la cartella dello stack, es. `stacks/<nome>/.env`).
- Nessun segreto nei file versionati (`docker-compose.yml`, `CLAUDE.md`, script, ecc.).
- `.env.example` documenta le variabili richieste, senza valori reali.
- I file `.env` sono in `.gitignore`.
