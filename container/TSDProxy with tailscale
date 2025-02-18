## Sources
* [https://almeidapaulopt.github.io/tsdproxy/](https://almeidapaulopt.github.io/tsdproxy/)
* [https://www.youtube.com/watch?v=5lJrXEXF8eM](https://www.youtube.com/watch?v=5lJrXEXF8eM)
* [https://github.com/tailscale-dev/video-code-snippets/blob/main/2024-11-tsdproxy/compose.yaml](https://github.com/tailscale-dev/video-code-snippets/blob/main/2024-11-tsdproxy/compose.yaml)

## Reverse Proxy setup
In your tailscale console [https://login.tailscale.com/admin/dns](https://login.tailscale.com/admin/dns):
- Set a tailnet name
- enable MagicDNS
- enable HTTPS certificates

In your tailscale console [https://login.tailscale.com/admin/machines/new-linux](https://login.tailscale.com/admin/machines/new-linux):
- Set up an auth key with 90 day expiry 

On your server, create a bridge network, container folder structure and docker-compose.yml
``` bash
docker network create -d bridge tailscalebridge
mkdir -p /volume1/docker/tsdproxy/ && cd /volume1/docker/tsdproxy
mkdir data
mkdir config
vi docker-compose.yml
```

``` yaml
services:
  tsdproxy:
    image: almeidapaulopt/tsdproxy:latest
    container_name: tsdproxy
    hostname: tsdproxy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /volume1/docker/tsdproxy/data:/data
      - /volume1/docker/tsdproxy/config:/config
    environment:
      # Get AuthKey from your Tailscale account
      - TSDPROXY_AUTHKEY=tskey-auth-kC43EMqn1a11CNTRL-notarealkey
      - DOCKER_HOST=unix:///var/run/docker.sock
    restart: unless-stopped
networks:
  default:
    external: true
    name: tailscalebridge
```

``` bash
docker compose up -d
```

## Example Docker: it-tools
``` bash
mkdir -p /volume1/docker/ittools && cd /volume1/docker/ittools
vi docker-compose.yml
```

``` bash
services:
  ittools:
    image: corentinth/it-tools
    container_name: ittools
    hostname: ittools
    labels:
      tsdproxy.enable: "true"
      tsdproxy.name: "ittools"
      tsdproxy.container_port: 8267
    ports:
      - 8267:80
    restart: unless-stopped
networks:
  default:
    external: true
    name: tailscalebridge
```
