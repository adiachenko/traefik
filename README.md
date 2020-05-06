This repository provides preconfigured instance of Traefik to run alongside your dockerized projects as an edge router in development. Using Traefik, we can easily use URLs like `https://example.test`, `https://api.example.test` or even `https://example.test/api/` to more accurately replicate our production setup locally.

Clone this repository to anywhere in your system and cd into it

> It is recommended to use one Traefik instance per machine as opposed to having separate Traefik instance for each dockerized repo since you will typically orchestrate several related projects (e.g. web, api, admin, etc.).

```sh
git clone https://github.com/adiachenko/traefik.git
cd traefik
```

Install mkcert

```sh
brew install mkcert nss
```

Install the local CA in the system trust store (circumvents security warnings when using self-signed certificates)

```sh
mkcert -install
```

Generate SSL certificates

```sh
mkcert example.test "*.example.test"
```

Move the generated certificates under `config/certificates` folder

Next, create certificates.yml file

```sh
cp config/certificates.yml.example config/certificates.yml
```

Edit the file to reference the newly added certificates.

You can run Traefik now

```sh
docker-compose up -d
```

Finally, we need to configure our projects to be discoverable by Traefik.

Add the following options to your project's `docker-compose.yml` file. Edit domains and path prefixes as desired (must match domains you created certificates for)

```yml
services:
  webserver:
    # ...
    labels:
      - "traefik.http.routers.nginx.rule=HostRegexp(`example.test`,`{subdomain:.+}.example.test`) && PathPrefix(`/`)"
      - "traefik.http.routers.nginx.entrypoints"
      - "traefik.http.routers.nginx.tls=true"
      - "traefik.http.routers.nginx.tls.domains[0].main=example.test"
      - "traefik.http.routers.nginx.tls.domains[0].sans=*.example.test"
    networks:
      - default
      - traefik

networks:
  traefik:
    external:
      name: traefik
```

Restart your Docker Compose services. You should be able to open your site via [https:://example.test](https:://example.test) (or whatever URL you chose previously).