# \[PT-BR\] searxng-docker

Crie uma instância do SearXNG em cinco minutos usando o Docker.

## Quais serviços são usados?

| Name | Description | Docker image | Dockerfile |
| --- | --- | --- | --- |
| [Caddy](https://github.com/caddyserver/caddy) | Proxy reverso (cria um certificado LetsEncrypt automaticamente) | [caddy/caddy:2-alpine](https://hub.docker.com/_/caddy) | [Dockerfile](https://github.com/caddyserver/caddy-docker) |
| [SearXNG](https://github.com/searxng/searxng) | O próprio SearXNG | [searxng/searxng:latest](https://hub.docker.com/r/searxng/searxng) | [Dockerfile](https://github.com/searxng/searxng/blob/master/Dockerfile) |
| [Redis](https://github.com/redis/redis) | Banco de dados em memória | [redis:alpine](https://hub.docker.com/_/redis) | [Dockerfile-alpine.template](https://github.com/docker-library/redis/blob/master/Dockerfile-alpine.template) |

## Como instalar?

- [Instale o docker](https://docs.docker.com/install/)
    
- [Instale o docker-compose](https://docs.docker.com/compose/install/) (be sure that docker-compose version is at least 1.9.0)
    
- Baixe o repositório searxng-docker. Exemplo:
    
    ```sh
    cd /usr/local
    git clone https://github.com/searxng/searxng-docker.git
    cd searxng-docker
    ```
    
- Edite o arquivo [.env](https://github.com/searxng/searxng-docker/blob/master/.env) e coloque um hostname e um email
    
- Crie uma chave secreta `sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml`
    
- Personalize o arquivo [searxng/settings.yml](https://github.com/searxng/searxng-docker/blob/master/searxng/settings.yml) (opcional)
    
- Verifique se tudo está funcionando: `docker-compose up`
    
- Rode o SearXNG em background: `docker-compose up -d`
    

## Como acessar os logs

Para acessar os logs de todos os containers use: `docker-compose logs -f`.

Para acessar os logs de um container específico:

- Caddy: `docker-compose logs -f caddy`
- SearXNG: `docker-compose logs -f searxng`
- Redis: `docker-compose logs -f redis`

### Iniciar SearXNG com systemd

Você pode pular esse passo se você não usar o systemd.

- `cp searxng-docker.service.template searxng-docker.service`
    
- edite o conteúdo do `WorkingDirectory` no arquivo `searxng-docker.service` (somente se o caminho de instalação for diferente de /usr/local/searxng-docker)
    
- Instale o systemd unit:
    
    ```sh
    systemctl enable $(pwd)/searxng-docker.service
    systemctl start searxng-docker.service
    ```
    

## Nota sobre o recurso de proxy de imagem

O proxy de imagem SearXNG é ativado por padrão.

O padrão [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) permite ao navegador acessar o `${SEARXNG_HOSTNAME}` e o `https://*.tile.openstreetmap.org;`.

Se você quiser desativar o proxy de imagem, é necessário modificar o [./Caddyfile](https://github.com/searxng/searxng-docker/blob/master/Caddyfile). Substitua o `img-src 'self' data: https://*.tile.openstreetmap.org;` by `img-src * data:;`.

## Multi Architecture Docker images

Arquiteturas suportadas:

- amd64
- arm64
- arm/v7

## Como faço para atualizar os containers?

Para atualizar a stack do SearXNG, digite:

```sh
docker-compose pull
docker-compose down
docker-compose up
```

Para fazer o update do arquivo `docker-compose.yml`:

Confira as atualizações do SearXNG no repositório oficial no GitHub: [searxng/searxng-docker](https://github.com/searxng/searxng-docker).

* * *
# \[EN-US\] searxng-docker

Create a new SearXNG instance in five minutes using Docker

## What is included ?

| Name | Description | Docker image | Dockerfile |
| -- | -- | -- | -- |
| [Caddy](https://github.com/caddyserver/caddy) | Reverse proxy (create a LetsEncrypt certificate automatically) | [docker.io/library/caddy:2-alpine](https://hub.docker.com/_/caddy)           | [Dockerfile](https://github.com/caddyserver/caddy-docker/blob/master/Dockerfile.tmpl) |
| [SearXNG](https://github.com/searxng/searxng) | SearXNG by itself                                              | [docker.io/searxng/searxng:latest](https://hub.docker.com/r/searxng/searxng) | [Dockerfile](https://github.com/searxng/searxng/blob/master/Dockerfile)               |
| [Valkey](https://github.com/valkey-io/valkey) | In-memory database                                             | [docker.io/valkey/valkey:8-alpine](https://hub.docker.com/r/valkey/valkey)        | [Dockerfile](https://github.com/valkey-io/valkey-container/blob/mainline/Dockerfile.template)             |

## How to use it
There are two ways to host SearXNG. The first one doesn't require any prior knowledge about self-hosting and thus is recommended for beginners. It includes caddy as a reverse proxy and automatically deals with the TLS certificates for you. The second one is recommended for more advanced users that already have their own reverse proxy (e.g. Nginx, HAProxy, ...) and probably some other services running on their machine. The first few steps are the same for both installation methods however.

1. [Install docker](https://docs.docker.com/install/)
2. Get searxng-docker
  ```sh
  cd /usr/local
  git clone https://github.com/searxng/searxng-docker.git
  cd searxng-docker
  ```
3. Edit the [.env](https://github.com/searxng/searxng-docker/blob/master/.env) file to set the hostname and an email
4. Generate the secret key `sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml`
5. Edit [searxng/settings.yml](https://github.com/searxng/searxng-docker/blob/master/searxng/settings.yml) according to your needs

> [!NOTE]
> On the first run, you must remove `cap_drop: - ALL` from the `docker-compose.yaml` file for the `searxng` service to successfully create `/etc/searxng/uwsgi.ini`. This is necessary because the `cap_drop: - ALL` directive removes all capabilities, including those required for the creation of the `uwsgi.ini` file. After the first run, you should re-add `cap_drop: - ALL` to the `docker-compose.yaml` file for security reasons.

> [!NOTE]
> Windows users can use the following powershell script to generate the secret key:
> ```powershell
> $randomBytes = New-Object byte[] 32
> (New-Object Security.Cryptography.RNGCryptoServiceProvider).GetBytes($randomBytes)
> $secretKey = -join ($randomBytes | ForEach-Object { "{0:x2}" -f $_ })
> (Get-Content searxng/settings.yml) -replace 'ultrasecretkey', $secretKey | Set-Content searxng/settings.yml
> ```

### Method 1: With Caddy included (recommended for beginners)
6. Run SearXNG in the background: `docker compose up -d`

### Method 2: Bring your own reverse proxy (experienced users)
6. Remove the caddy related parts in `docker-compose.yaml` such as the caddy service and its volumes.
7. Point your reverse proxy to the port set for the `searxng` service in `docker-compose.yml` (8080 by default).
8. Generate and configure the required TLS certificates with the reverse proxy of your choice.
9. Run SearXNG in the background: `docker compose up -d`

> [!NOTE]
> You can change the port `searxng` listens on inside the docker container (e.g. if you want to operate in `host` network mode) with the `BIND_ADDRESS` environment variable (defaults to `0.0.0.0:8080`). The environment variable can be set directly inside `docker-compose.yaml`.

## Troubleshooting - How to access the logs

To access the logs from all the containers use: `docker compose logs -f`.

To access the logs of one specific container:

- Caddy: `docker compose logs -f caddy`
- SearXNG: `docker compose logs -f searxng`
- Valkey: `docker compose logs -f redis`

### Start SearXNG with systemd

You can skip this step if you don't use systemd.

- ```cp searxng-docker.service.template searxng-docker.service```
- edit the content of ```WorkingDirectory``` in the ```searxng-docker.service``` file (only if the installation path is different from /usr/local/searxng-docker)
- Install the systemd unit:
  ```sh
  systemctl enable $(pwd)/searxng-docker.service
  systemctl start searxng-docker.service
  ```

## Note on the image proxy feature

The SearXNG image proxy is activated by default.

The default [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) allow the browser to access to ```${SEARXNG_HOSTNAME}``` and ```https://*.tile.openstreetmap.org;```.

If some users want to disable the image proxy, you have to modify [./Caddyfile](https://github.com/searxng/searxng-docker/blob/master/Caddyfile). Replace the ```img-src 'self' data: https://*.tile.openstreetmap.org;``` by ```img-src * data:;```.

## Multi Architecture Docker images

Supported architecture:

- amd64
- arm64
- arm/v7

## How to update ?

To update the SearXNG stack:

```sh
git pull
docker compose pull
docker compose up -d
```

Or the old way (with the old docker-compose version):

```sh
git pull
docker-compose pull
docker-compose up -d
```
