# Symfony Docker (Scalable Edition)

A [Docker](https://www.docker.com/)-based installer and runtime for the [Symfony](https://symfony.com) web framework,
with [FrankenPHP](https://frankenphp.dev), [Traefik](https://traefik.io/), and a standalone [Mercure](https://mercure.rocks) hub!

![CI](https://github.com/dunglas/symfony-docker/workflows/CI/badge.svg)

**⚡ This fork adds horizontal scaling capabilities with Traefik as a reverse proxy and a decoupled Mercure hub.**

## Getting Started

1. If not already done, [install Docker Compose](https://docs.docker.com/compose/install/) (v2.10+)
2. Run `docker compose build --pull --no-cache` to build fresh images
3. Run `docker compose up --wait` to set up and start a fresh Symfony project
4. Open `http://app.localhost` in your favorite web browser
5. Run `docker compose down --remove-orphans` to stop the Docker containers.

### Scaling Instances

Scale your FrankenPHP application horizontally:

```bash
# Scale to 3 instances
docker compose up --scale php=3 -d

# Scale to 5 instances
docker compose up --scale php=5 -d

# Check running instances
docker compose ps
```

See [SCALING.md](SCALING.md) for detailed documentation on the scalable architecture.

## Features

- **🚀 Horizontal Scaling**: Scale from 1 to N FrankenPHP instances with `--scale`
- **⚖️ Load Balancing**: Automatic distribution via Traefik with health checks
- **🔄 Decoupled Mercure**: Standalone hub shared by all instances
- **⚡ Worker Mode**: Blazing-fast performance thanks to [FrankenPHP worker mode](https://frankenphp.dev/docs/worker/)
- **🔒 Automatic HTTPS**: Let's Encrypt integration via Traefik (production)
- **📊 Monitoring**: Traefik dashboard for real-time metrics
- **🎯 Zero-downtime**: Rolling updates with health checks
- Production, development and CI ready
- HTTP/3 and [Early Hints](https://symfony.com/blog/new-in-symfony-6-3-early-hints) support
- Real-time messaging thanks to [Mercure hub](https://symfony.com/doc/current/mercure.html)
- [Vulcain](https://vulcain.rocks) support
- Native [XDebug](docs/xdebug.md) integration
- Super-readable configuration

**Enjoy!**

## Docs

1. [🚀 Horizontal Scaling & Architecture](SCALING.md) **← New!**
2. [Options available](docs/options.md)
3. [Using Symfony Docker with an existing project](docs/existing-project.md)
4. [Support for extra services](docs/extra-services.md)
5. [Deploying in production](docs/production.md)
6. [Debugging with Xdebug](docs/xdebug.md)
7. [TLS Certificates](docs/tls.md)
8. [Using MySQL instead of PostgreSQL](docs/mysql.md)
9. [Using Alpine Linux instead of Debian](docs/alpine.md)
10. [Using a Makefile](docs/makefile.md)
11. [Updating the template](docs/updating.md)
12. [Troubleshooting](docs/troubleshooting.md)

## License

Symfony Docker is available under the MIT License.

## Credits

Created by [Kévin Dunglas](https://dunglas.dev), co-maintained by [Maxime Helias](https://twitter.com/maxhelias) and sponsored by [Les-Tilleuls.coop](https://les-tilleuls.coop).
