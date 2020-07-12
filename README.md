# Home Server
Scripts collection to run dockerized set of home apps behind `Traefik` reverse proxy.

Everything should work without any pre-configuration, except few things:
* self-signed certificate, which must be created before hand with
    ```shell script
    cd traefik && openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/private.key -x509 -days 3650 -out certs/cert.crt 
    ```
* external docker network with the name `web`:
    ```shell script
    docker network create web
    ```
* Update mounted directories with storage location (replace `/media/8tb` with your path)

Then you can run everything with `docker-compose`:
```shell script
cd traefik && docker-compose up -d
cd web-applications && docker-compose up -d
```
or use `systemd` units: TBD