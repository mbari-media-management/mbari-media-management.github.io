# Microservices

## Quick Start

1. Install [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/).
2. Clone the [m3-microservices](https://github.com/mbari-media-management/m3-microservices) repository and build the images:

```bash
git clone https://github.com/mbari-media-management/m3-microservices.git
cd m3-microservices

# Edit .env file and set M3_HOST_DIR to the full path to the 
# temp dir under m3-microservices

docker-compose build
docker-compose up
```

You will now have the VARS/M3 microservices up and running on your machine. This can be used across you entire internal network and is robust enough for modest deployments. All data will be persisted to a local database under `m3-microservices/derby`

## Customizing Setup

It's easy to configure the services for your own database. The detailed docs will be _coming real soon ..._