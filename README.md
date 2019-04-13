# nodelol
Deploy auto-updating, HTTPS wildcard certificate, reverse-proxied containers with ease

## Getting Started
These instructions will cover the setup, config and usage for the containers

### Prerequisites
* `docker`
* `docker-compose`
* Domain that you control
* [CloudFlare](https://www.cloudflare.com) free-tier account
* [Docker Hub](https://hub.docker.com) account

#### Install and Configure Prerequisites
##### docker 
https://docs.docker.com/get-started/

##### docker-compose
https://docs.docker.com/compose/install/

##### CloudFlare
Configure your domain to use CloudFlare DNS, wait for propagation and retrieve the CloudFlare Global API Key from your profile page

### Usage
##### Configure Environment Variables
Create and edit `.env`
```shell
cp .env.example .env
nano .env
```
Add values to the environment variables
* `DOMAIN` - the root domain configured with CloudFlare DNS
* `CLOUDFLARE_EMAIL` - your email registered with CloudFlare
* `CLOUDFLARE_API_KEY` - your CloudFlare global API key

#### traefik Configuration

[traefik](https://github.com/containous/traefik) is used for automatically requesting and renewing the LetsEncrypt wildcard certificate and as the reverse-proxy for your containers. As you deploy more containers, traefik will make it accessible as an HTTPS-protected sub-domain.

First, using the `htpasswd` utility, create an HTTP basic authentication token:

```shell
apt install apache2-utils
htpasswd -nb admin your_password
```

Create and edit the `traefik.toml` file:

```shell
cp ./etc/traefik/traefik.toml.example ./etc/traefik/traefik.toml
nano ./etc/traefik/traefik.toml
```

Replace "HTPASSWD BASIC AUTH" with the token generated with `htpasswd`.

Replace "CERTIFICATE EMAIL ADDRESS" with an email that will be used for your HTTPS certificate.

Replace "DOMAIN" with your root domain configured with CloudFlare DNS.

Create and edit permissions on the `acme.json` file where traefik will store LetsEncrypt certificate data:
```shell
touch ./etc/acme/acme.json
chmod 600 ./etc/acme/acme.json
```

#### watchtower Configuration

Login to Docker Hub and create a symlink to the `config.json` file which is used by watchtower:
```shell
docker login
ln -s ~/.docker/config.json ./etc/watchtower/config.json
```

#### docker Configuration

Create the 'web' network which traefik and all traefik-proxied containers will use:
```shell
docker network create web
```

The `docker-compose.yml` file is used to create the containers for this project and comes with:
* [watchtower](https://github.com/containrrr/watchtower) (updates monitored containers image to latest)
* [traefik](https://github.com/containous/traefik) (reverse-proxy for containers)
* [portainer](https://github.com/portainer/portainer) (web-interface for docker container management)

Create containers:
```shell
docker-compose up -d
```

#### Post-Setup Config and Verification
Verify all is working:

##### traefik
* Navigate to: `https://traefik.yourdomain.tld`
* You should see an HTTP basic auth window
* Enter your 'admin' as user and the password you used with `htpasswd`
* Confirm the HTTPS cert is valid and shows that it is issued to `*.yourdomain.tld`

##### portainer
* Navigate to: `https://portainer.yourdomain.tld`
* Create an administrator password and sign in
* Choose "local" and continue to the dashboard
* Navigate to the containers page and see that all 3 containers are in a green "running" state

##### watchtower
* Select the 'watchtower' container in the portainer dashboard
* Click on the 'Logs' button and confirm there are no errors


