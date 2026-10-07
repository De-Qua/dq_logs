# DeQua Logs

Servizio per consultare da browser i log di tutti i container Docker di DeQua, tramite [Dozzle](https://dozzle.dev).

Dozzle legge i log direttamente dal socket Docker: non serve configurare niente nei singoli servizi, basta che scrivano su stdout/stderr. Vengono mostrati i log ancora presenti in Docker (per ogni servizio sono configurati 5 file da 10 MB); i container rimossi, ad esempio quelli lanciati con `docker compose run --rm`, spariscono insieme ai loro log.

## Configurazione

Le impostazioni sono nel file `.env`. Prima di avviare il servizio create un file `.env` a partire da `.env.example`:

```bash
cp .env.example .env
```

Di default è attiva l'autenticazione (`DOZZLE_AUTH_PROVIDER=simple`), mentre le azioni sui container e l'accesso alla shell sono disattivati. Lasciateli disattivati: chi accede a Dozzle potrebbe fermare i container o aprirci dentro una shell.

## Creare gli utenti

Gli utenti sono nel file `dozzle_data/users.yml` (dentro il container è `/data/users.yml`). La cartella `dozzle_data/` è esclusa da git perché contiene gli hash delle password.

Per generare un utente:

```bash
mkdir -p dozzle_data
docker run --rm amir20/dozzle generate admin \
  --password 'UNA-PASSWORD-ROBUSTA' \
  --email nome@dominio.it \
  --name "Admin" \
  > dozzle_data/users.yml
```

Il comando scrive un file come questo, con la password salvata come hash bcrypt:

```yaml
users:
    admin:
        email: nome@dominio.it
        name: Admin
        password: $2a$11$...
        github: ""
        filter: ""
        roles: ""
```

Per aggiungere altri utenti generateli allo stesso modo su un file temporaneo e copiate la loro voce sotto `users:` in `dozzle_data/users.yml`. Attenzione a non usare `>` sul file esistente, altrimenti lo sovrascrivete.

Per cambiare una password rigenerate l'utente e sostituite la sua voce. Dopo ogni modifica di `users.yml` riavviate il servizio:

```bash
docker compose restart
```

## Avvio

In produzione la porta non è pubblicata sul server: Dozzle è raggiungibile solo dalla rete Docker `dequa_nw`, all'indirizzo `http://dq_logs:8080`. Per accedere da browser serve un reverse proxy con HTTPS su quella rete (con HTTP semplice la password viaggerebbe in chiaro).

```bash
docker compose up -d
```

In sviluppo `docker-compose.dev.yml` pubblica la porta 8080, quindi Dozzle è raggiungibile su http://localhost:8080:

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

Se pubblicate la porta su un server, limitatela a localhost (`"127.0.0.1:8080:8080"`) e accedete tramite tunnel SSH o VPN: le porte pubblicate da Docker non vengono bloccate da `ufw`.
