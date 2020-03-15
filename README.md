# studentenportal.ch 2020 deployment

The Studentenportal deployment currently deployed on studentenportal.ch,
replacing the older "deploy" and "deploy-new" repositories.

## Bird's eye view

There are three [Docker](https://www.docker.com/) containers running,
orchestrated using [docker-compose](https://docs.docker.com/compose/):

- **nginx**, containing a [nginx](http://nginx.org/) webserver as reverse proxy
  and [dehydrated](https://github.com/dehydrated-io/dehydrated) for [Let's
  Encrypt](https://letsencrypt.org/) HTTPS certificates. Static files are also
  served directly via nginx.
- **postgres**, containing a [PostgreSQL](https://www.postgresql.org/) database.
- **studentenportal**, containing the
  [studentenportal/web](https://github.com/studentenportal/web) repository and
  running [Gunicorn](https://gunicorn.org/).

Other than Docker, only a few things are running on the host directly:

- [UFW](https://help.ubuntu.com/community/UFW) as firewall
- systemd timer to refresh certificates by running dehydrated via `docker exec`
- [msmtp](https://marlam.de/msmtp/) so the host can send mails via
  `team@studentenportal.ch`. Currently, Docker containers can't send mails and
  Django uses SMTP directly. It's planned to set up a proper MTA to fix this.
  
## Logging into the server

To log in to the server behind studentenportal.ch, ssh to
`root@studentenportal.ch`. Most services run as the `studentenportal` user which
has `nologin` as shell, so you'll need to `su studentenportal -s /bin/bash`.

## Relevant files

All relevant data is in `/home/studentenportal` on the server.

- `~/media` is the Django media folder, mapped to
  `/srv/www/studentenportal/media` in the `nginx` and `studentenportal`
  containers.
- `~/postgres-data` is mapped to `/var/lib/postgresql/data` in the `postgres`
  container.
- `~/studentenportal.env` is the docker environment file. It's deployed via
  Ansible and needs to set:
  * POSTGRES_DB_NAME (e.g. studentenportal)
  * POSTGRES_USER (e.g. studentenportal)
  * POSTGRES_PASSWORD (e.g. hunter2)
  * SECRET_KEY (for Django)
  * DJANGO_EMAIL_HOST (with STARTTLS)
  * DJANGO_EMAIL_HOST_USER
  * DJANGO_EMAIL_HOST_PASSWORD
- `~/web` is the [studentenportal/web](https://github.com/studentenportal/web)
  repository.
- Static files are stored in a `studentenportal-static` named docker volume and
  not mapped to the host.
- Dehydrated data (account/certificate) is stored in a
  `studentenportal-dehydrated` named docker volume and not mapped to the host.
  
## Repositories

This repository contains [Ansible](https://www.ansible.com/) configurations to
set up the host machine and sets up the certificate inside the Docker container.

The private `pass` repository contains passwords needed to run Ansible.
  
The `web` repository contains `docker-compose-production.yml` which sets up the
production environment. It uses data from `deploy/production/` in the same
repository, including the nginx/dehydrated configuration. This is so that it's
possible to simulate the real deployment locally. Note that the `web` repository
sets up nginx with a self-signed snakeoil certificate, which then gets replaced
by a proper one by running `dehydrated` via Ansible.`

## Deploying

To deploy, do the following:

- Make sure you can access the server via SSH using key-based authentication.
- Clone the "pass" repository so it's inside this repository under pass/
- Run `ansible-playbook site.yml`
