# Infrastructure

> Configurations for all of my services.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](https://opensource.org/licenses/MIT) [![Pipeline (config-check)](https://img.shields.io/github/workflow/status/pascaliske/infrastructure/Config%20Check/master?label=config-check&style=flat-square)](https://github.com/pascaliske/infrastructure/actions) [![Pipeline (linting)](https://img.shields.io/github/workflow/status/pascaliske/infrastructure/Linting/master?label=linting&style=flat-square)](https://github.com/pascaliske/infrastructure/actions) [![GitHub Last Commit](https://img.shields.io/github/last-commit/pascaliske/infrastructure?style=flat-square)](https://github.com/pascaliske/infrastructure)

This repository contains the configurations for most of my services:

- [Authelia](https://www.authelia.com) as SSO platform for all services
- [Blocky](https://0xerr0r.github.io/blocky/) for blocking ads and malicious domains network-wide
- [cert-manager](https://cert-manager.io) as automated certificate management tool
- [Cloudflared](https://github.com/cloudflare/cloudflared) for securing all DNS traffic using [DNS-over-HTTPS](https://en.m.wikipedia.org/wiki/DNS_over_HTTPS)
- [Code Server](https://github.com/cdr/code-server), an in-browser VS Code instance
- [GitLab](https://about.gitlab.com/) as DevOps platform (including GitLab Runner)
- [Grafana](https://grafana.com/) for querying and displaying metrics data
- [Home Assistant](https://home-assistant.io), an Home Automation platform
- [Homer](https://github.com/bastienwirtz/homer) as service dashboard
- [Keel](https://keel.sh) for auto updating all services
- [Linkding](https://github.com/sissbruecker/linkding), an self-hosted bookmark service
- [Paperless](https://github.com/jonaswinkler/paperless-ng), an document index and management platform
- [Prometheus](https://prometheus.io) for collecting service metrics
- [Shlink](https://shlink.io), a self-hosted link shortener
- [Snapdrop](https://github.com/RobinLinus/snapdrop), local file sharing inspired by Apple's AirDrop
- [Traefik](https://traefik.io) reverse proxy for all services
- [UniFi Controller](https://www.ui.com/software/) for managing all UniFi network gear
- [Uptime Kuma](https://github.com/louislam/uptime-kuma), fancy self-hosted monitoring tool
- [Vaultwarden](https://github.com/dani-garcia/vaultwarden), self-hosted password management tool
- [Velero](https://velero.io) for scheduled backups of all services

## Requirements

- [Node.js](https://nodejs.org/) + [Yarn](https://yarnpkg.com)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [K3s](https://rancher.com/docs/k3s/latest/en/)

## Install

```zsh
# clone the repo to your local machine
git clone https://github.com/pascaliske/infrastructure

# install needed dependencies
yarn install

# provision target zone using ansible
yarn run play playbooks/{zone}/configure.yml

# bootstrap flux
flux bootstrap github \
    --owner=pascaliske \
    --repository=infrastructure \
    --branch=feature/flux \
    --path=./cluster/base \
    --personal

# ssh into target host from inventory
yarn run ssh {host}
```

## Update

Keel automatically checks for updates of all deployments every night.
To manually update a deployment you can use the following commands:

```zsh
# ssh into target host from inventory
yarn run ssh {host}

# pull image updates
sudo ctr image pull {image} # e.g. docker.io/alpine:latest

# restart deployment
kubectl rollout restart -n {namespace} deployment/{app}
```

## Service CLIs

### The `redis-cli` command

```zsh
kubectl exec -it -n redis deploy/redis -- redis-cli # interactive
kubectl exec -it -n redis deploy/redis -- redis-cli <command> # one-off
```

For more information on the `redis-cli` command itself [visit their docs](https://redis.io/topics/rediscli).

### The `gitlab-backup` command

```zsh
kubectl exec -it -n gitlab deploy/gitlab -- gitlab-backup <task> # tasks: create | restore
```

For more information on the `gitlab-backup` command itself [visit their docs](https://docs.gitlab.com/ee/raketasks/backup_restore.html#back-up-gitlab).

### The `blocky` command

```zsh
kubectl exec -it -n blocky deploy/blocky -- ./blocky <command>
```

For more information on the `blocky` command itself [visit their docs](https://0xerr0r.github.io/blocky/interfaces/).

### The `hass` command

```zsh
kubectl exec -it -n home-assistant deploy/home-assistant -- hass -h
```

For more information on the `hass` command itself [visit their docs](https://www.home-assistant.io/docs/tools/hass/).

### The `paperless` management utilities

```zsh
kubectl exec -it -n paperless deploy/paperless -- document_exporter
kubectl exec -it -n paperless deploy/paperless -- document_importer
kubectl exec -it -n paperless deploy/paperless -- document_retagger
```

For more information on the commands itself [visit their docs](https://paperless-ng.readthedocs.io/en/latest/administration.html#management-utilities).

### The `shlink` command

```zsh
kubectl exec -it -n shlink deploy/shlink -- shlink short-url:list [--tags=<tag1>,<tag2>]
kubectl exec -it -n shlink deploy/shlink -- shlink short-url:generate <url> [--custom-slug=<slug>]
kubectl exec -it -n shlink deploy/shlink -- shlink short-url:import <source>
```

For more information on the commands itself [visit their docs](https://shlink.io/documentation/command-line-interface/).

## Hardware (Group Network)

The network services are currently running inside a K3s cluster on a [Raspberry Pi 4 Model B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/). It has the [official Raspberry Pi PoE-Hat](https://www.raspberrypi.org/products/poe-hat/) attached which powers it using the `802.3af` Power-over-Ethernet standard.

The fan of the PoE hat appears to be very noisy. Therefore I adjusted the temperature thresholds of the fan inside of `/boot/config.txt` to 70°C and 80°C:

```toml
# Fan settings from the official RPi PoE-Hat
dtparam=poe_fan_temp0=70000,poe_fan_temp0_hyst=5000
dtparam=poe_fan_temp1=80000,poe_fan_temp1_hyst=2000
```

## License

[MIT](LICENSE.md) – © 2022 [Pascal Iske](https://pascaliske.dev)
