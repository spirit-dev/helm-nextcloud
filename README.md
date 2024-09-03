# Welcome to nextcloud

[![App Status](https://argocd-internal.spirit-dev.net/api/badge?name=nextcloud-turingpi&revision=true&showAppName=true)](https://argocd-internal.spirit-dev.net/applications/nextcloud-turingpi)

## Table of content

- [Welcome to nextcloud](#welcome-to-nextcloud)
  - [Table of content](#table-of-content)
  - [Installation process](#installation-process)
  - [Run manual command](#run-manual-command)

## Installation process

The installation is entirely managed by Argocd.

A `Makefile` is present here to ease the first and one-time deployment or in case of an issue.
The installation should be done in two steps:

```shell
#> make dry-run ENV=<ENV>
#> make install ENV=<ENV>
```

## Run manual command

If you need to jump in the pod and run some manual commands. This can be done with the following:

```bash
#> runuser --user www-data -- php occ upgrade/other

Can also be:

#> kubectl exec -n [ns] [pod] nextcloud -- runuser --user www-data -- php occ upgrade/other

```
