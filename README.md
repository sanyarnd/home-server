# Home Server
Scripts collection to run dockerized set of home apps behind `Traefik` reverse proxy.

Application list:
* Traefik - routing and encryption (`/dashboard/`)
* Watchtower - keep images updated
* Jellyfin - media content (`/jellyfin`)
* MiniDLNA (local network only, `8200` port)
* Transmission - torrents (`/torrent`)
* Portainer - docker monitoring (`/portainer`)
* Filebrowser - access mounted filesystem from browser (`/filebrowser`)

Everything should work without any pre-configuration, except a few things:
* self-signed certificate, which must be created before hand with
    ```shell script
    cd traefik && openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/traefik.key -x509 -days 3650 -out certs/traefik.crt 
    ```
* external docker network with the name `web`:
    ```shell script
    docker network create web
    ```
* Update mounted directories with storage location (replace `/media/8tb` with your own)
* Initial `Jellyfin` setup: by default docker-compose exposes `Jellyfin`'s `8096` port. Use it to connect to the server (`http://<server ip>:8096`) go though initial configuration, and then go to settings and update default base url in admin panel. Then you can remove port exposure, if you want. There is no other way to change base url.
* Some services (e.g. `Transmission`) do not allow to set base url, so they'll fail to redirect to correct URL after authentication. Simply fix URL (add missing path element)

Then you can run everything with `docker-compose`:
```shell script
cd watchtower && docker-compose up -d
cd traefik && docker-compose up -d
cd web-applications && docker-compose up -d
```
