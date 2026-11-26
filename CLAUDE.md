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
- Traefik è l'unico stack che pubblica porte (80) sull'host.

## Immagini

- Ogni immagine ha una versione fissa (es. `traefik:v3.1.6`).
- Mai usare `latest` né tag senza versione.

## Segreti

- I segreti stanno solo nel file `.env` (dentro la cartella dello stack, es. `stacks/<nome>/.env`).
- Nessun segreto nei file versionati (`docker-compose.yml`, `CLAUDE.md`, script, ecc.).
- `.env.example` documenta le variabili richieste, senza valori reali.
- I file `.env` sono in `.gitignore`.
