# Wordpress + Traefik + Let's Encrypt certificates using Docker

This repository is all you need to have your own Wordpress site (or sites) running behind [Traefik](https://docs.traefik.io/v1.0/) proxy with free autorenewal wildcard certificate from [Let's Encrypt](https://letsencrypt.org/). And all of it with the power of [Docker](https://www.docker.com/) and the scalability provided by [Docker Swarm](https://docs.docker.com/engine/swarm/).

![Traefik](https://docs.traefik.io/img/architecture.png)

## What you need
* **ONE** server, i.e. an EC2 instance on AWS.
* **ONE** MySQL database, i.e. an RDS instance on AWS. However, this is not really mandatory as you can have another container that runs MySQL. See the official [Worpress Docker Hub](https://hub.docker.com/_/wordpress/) image for instructions.
* Docker installed on the server.
* A domain registered, i.e. Route53.

### For local development
* [Docker Compose](https://docs.docker.com/compose/).

## Installation
Clone this repository and change the files accordingly with the appropriate values, like:
* Domain.
* Passwords.
* Docker network, volumes and secrets.
* Wordpress configuration values: database host, name, username, etc.
* Traefik labels.

### Create the Docker virtual network
The Traefik proxy and the Worpress (or any other webapp) must be under the same virtual network. So, before running the containers create the network by running:

```bash
# For local development
docker network create <YOUR_NETWORK>

# For production (Swarm mode)
docker network create --driver overlay <YOUR_NETWORK>
```

### `traefik.toml`
This is the Traefik [configuration file](./traefik.toml) and it contains self-explanatory comments for each section.

For local development, the Staging CA Server from Let's Encrypt can be used. To do so, add the following line under `[acme]` section:

```toml
[acme]
...
caServer="https://acme-staging-v02.api.letsencrypt.org/directory"

```

For more information visit:
* https://docs.traefik.io/v1.0/toml/
* Examples: https://docs.traefik.io/user-guide/examples/
* Let's Encrypt configuration: https://docs.traefik.io/configuration/acme/

### `traefik-docker-compose.yml`
[This Compose file](./traefik-docker-compose.yml) creates a container (or stack/service if using Swarm) with Traefik proxy running and listening on ports 80 and 443 - although all traffic on port 80 is redirected to port 443. Then, in turns, the traffic is redirected to the appropriate containers with the applications using the `labels`.

Before running this file, update the values for:
* Docker network.
* Traefik labels.

This file makes the monitoring UI available at the URL defined in the `traefik.frontend.rule` label via HTTPS.

### `yourwp-docker-compose.yml`
[This Compose file](yourwp-docker-compose.yml) contains all the configuration required to create a container (or stack/service if using Swarm) with the selected version of Worpress.

Before running this file, update the values for:
* Version of Worpress.
* Wordpress environment.
* Docker network, volumes and secrets.
* Traefik labels.

### `acme.json`
This empty file will contain the certificates generated from Let's Encrypt once the containers have started. It needs specific file permissions for security so, once cloned, run:

```bash
sudo chmod 600 acme.json
```

### Start the containers
Once the Traefik configuration file and the Compose files have been modified accordingly, start the containers by running the following commands:

```bash
# ----- For local development -----
# Start Traefik container
docker-compose -f traefik-docker-compose.yml up
# Start your Wordpress container
docker-compose -f yourwp-docker-compose.yml up

# ----- For production (Swarm mode) -----
# Enable Swarm
docker swarm init
# Start Traefik container
docker stack deploy -c traefik-docker-compose.yml proxy
# Start your Wordpress container
docker stack deploy -c yourwp-docker-compose.yml yourwp
```

Note that you can start as many Wordpress (or other webapps) as you want by cloning the Compose file and modifying accordingly.

You can also use the power of Docker Swarm to create replicas of the Wordpress containers across multiple hosts to scale out.

## Check that it worked
Once the containers have started you can visit both the monitoring built-in Traefik dashboard and the Wordpress site.

![Dashboard](https://docs.traefik.io/img/web.frontend.png)

![Health](https://docs.traefik.io/img/traefik-health.png)

The following blogs are have been powered as described above:
* [Travel Everwanderer](https://travel.everwanderer.com).
* [Tech Everwanderer](https://tech.everwanderer.com).

## License
Code copyright 2018. Code released under [the MIT License](./LICENSE.txt).