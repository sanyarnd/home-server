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

## Reasoning

First, I wanted to re-use free DNS provided by `Mikrotik`. It's not pretty, but it's free, and it's possible to write a simple scheduled script for syncing (I have public IP, but it's dynamic).

Second, my ISP blocks all default ports (e.g. 22, 25, 80, 443, even 8080). It means I can't use `Let's Encrypt` and similar services (http challenges require any of 80/443 ports to be open, DNS challenge is not possible because I don't own domain). That's why I use self-signed certificate.

Third, I don't own domain, and I can't create subdomains, that's why reverse proxy uses path prefix middleware for navigation.

And lastly, it was a week long trial and error to find apps compatible with path prefix middleware, as well as working internally (local DNS) and externally (`Mikrotik` DNS).


## Installation

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

## Useful commands
Remote mounting:
```
sshfs -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3,cache_timeout=3600 -p <ssh-port> <user>@<url>:<server-path> <local-path>
```
Unmount:
```
fusermount -u <local-path>
```
