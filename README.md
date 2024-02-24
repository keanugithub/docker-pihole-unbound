# Pi-Hole + Unbound with DoT on Docker

[![Build and publish container](https://github.com/keanugithub/docker-pihole-unbound/actions/workflows/auto-build-container.yml/badge.svg)](https://github.com/keanugithub/docker-pihole-unbound/actions/workflows/auto-build-container.yml)
[![Docker Pulls](https://img.shields.io/docker/pulls/keanubdh/pihole-unbound)](https://hub.docker.com/r/kanubdh/pihole-unbound)


## Use Docker to run [Pi-Hole](https://pi-hole.net) with an upstream [Unbound](https://nlnetlabs.nl/projects/unbound/about/) resolver forwarding using [DoT](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-tls/)

- This already installs Unbound directly to the Pihole container.
- You only need to enable [DNS over TLS](https://wiki.archlinux.org/title/Domain_name_resolution#Privacy_and_security), if you want of course, by modifying `pihole-unbound/unbound-pihole.conf` on your favorite text editor and append the following configuration:
```
server:
...
	tls-system-cert: yes
...
forward-zone:
        name: "."
        forward-tls-upstream: yes
        forward-addr: 1.1.1.1@853#cloudflare-dns.com
```

NOTE: [For each server you will need to specify the connection port using @ and its domain name with #. The domain name is required for TLS authentication and also allows setting stub-zones and using the unbound-control forward control command with domain names. There should not be any spaces in the forward-addr specification.](https://wiki.archlinux.org/title/unbound)

If configured correctly, it should show something like [this](https://1.1.1.1/help#eyJpc0NmIjoiWWVzIiwiaXNEb3QiOiJZZXMiLCJpc0RvaCI6Ik5vIiwicmVzb2x2ZXJJcC0xLjEuMS4xIjoiWWVzIiwicmVzb2x2ZXJJcC0xLjAuMC4xIjoiWWVzIiwicmVzb2x2ZXJJcC0yNjA2OjQ3MDA6NDcwMDo6MTExMSI6Ik5vIiwicmVzb2x2ZXJJcC0yNjA2OjQ3MDA6NDcwMDo6MTAwMSI6Ik5vIiwiZGF0YWNlbnRlckxvY2F0aW9uIjoiTU5MIiwiaXNXYXJwIjoiTm8iLCJpc3BOYW1lIjoiQ2xvdWRmbGFyZSIsImlzcEFzbiI6IjEzMzM1In0=) on 1.1.1.1/help.

## Updates

This image is regularly updated with the latest release from the official pi-hole image.

Whenever there is an update for the [original pihole image](https://hub.docker.com/r/pihole/pihole), an automatic pull request is opened to implement the update. 

The workflow file for this can be found in `.github/workflows/auto-build-container.yml` which is scheduled to run everyday at 23:00 GitHub time.

This workflow runs when the image tag is updated in `pihole-unbound/Dockerfile` with the help of [the renovate bot](https://github.com/renovatebot/renovate). Hopefully, this minimizes the delay whenever there is an update for the [original pihole image](https://hub.docker.com/r/pihole/pihole)

## Description

This Docker deployment runs both Pi-Hole and Unbound in a single container.

The base image for the container is the [official Pi-Hole container](https://hub.docker.com/r/pihole/pihole), with an extra build step added to install the Unbound resolver directly into to the container based on [instructions provided directly by the Pi-Hole team](https://docs.pi-hole.net/guides/unbound/).

## Usage

First create a `.env` file to substitute variables for your deployment.

### Pi-hole environment variables

> Vars and descriptions replicated from the [official pihole container](https://github.com/pi-hole/docker-pi-hole/#environment-variables):

| Variable | Default | Value | Description |
| -------- | ------- | ----- | ---------- |
| `TZ` | UTC | `<Timezone>` | Set your [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) to make sure logs rotate at local midnight instead of at UTC midnight.
| `WEBPASSWORD` | random | `<Admin password>` | [http://pi.hole/admin](http://pi.hole/admin) password. Run `docker logs pihole \| grep random` to find your random pass.
| `FTLCONF_LOCAL_IPV4` | unset | `<Host's IP>` | Set to your server's LAN IP, used by web block modes and lighttpd bind address.
| `REV_SERVER` | `false` | `<"true"\|"false">` | Enable DNS conditional forwarding for device name resolution |
| `REV_SERVER_DOMAIN` | unset | Network Domain | If conditional forwarding is enabled, set the domain of the local network router |
| `REV_SERVER_TARGET` | unset | Router's IP | If conditional forwarding is enabled, set the IP of the local network router |
| `REV_SERVER_CIDR` | unset | Reverse DNS | If conditional forwarding is enabled, set the reverse DNS zone (e.g. `192.168.0.0/24`) |
| `WEBTHEME` | `default-light` | `<"default-dark"\|"default-darker"\|"default-light"\|"default-auto"\|"lcars">`| User interface theme to use. |
| `VIRTUAL_HOST` | `$FTLCONF_LOCAL_IPV4` | `<Custom Hostname>` | What your web server 'virtual host' is, accessing admin through this Hostname/IP allows you to make changes to the whitelist / blacklists in addition to the default ['http://pi.hole/admin'](http://pi.hole/admin) address |

Example `.env` file in the same directory as your `docker-compose.yaml` file:

```env
FTLCONF_LOCAL_IPV4=192.168.1.10
TZ=America/Los_Angeles
WEBPASSWORD=QWERTY123456asdfASDF
REV_SERVER=true
REV_SERVER_DOMAIN=local
REV_SERVER_TARGET=192.168.1.1
REV_SERVER_CIDR=192.168.0.0/16
HOSTNAME=pihole
DOMAIN_NAME=pihole.local
PIHOLE_WEBPORT=80
WEBTHEME=default-light
VIRTUAL_HOST=pihole.box
```

### Running the stack

```bash
docker-compose up -d
```

> If using Portainer, just paste the `docker-compose.yaml` contents into the stack config and add your *environment variables* directly in the UI.

## Building the image locally

- [ ] Clone this repo to you machine
- [ ] Run the commands below

```bash
cd docker-pihole-unbound
docker build . -t dev/docker-pihole-unbound:latest
```

## Automatic dev builds with Github Actions

There is a Github Action that runs on all pull requests that builds and publishes the image configured in the repo. The action can be found in `.github/workflows/dev-build.yml`. To use this feature please comment your repo and tag @keanugithub. I will run the workflow for you.

## Contributors

Thanks to all who made this project possible especially to the original creator [chriscrowe](https://github.com/chriscrowe/docker-pihole-unbound), and [aleksanderbl29](https://github.com/aleksanderbl29) for the basis of this fork.
