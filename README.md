# nextcloud

[![GitLab Sync](https://img.shields.io/badge/gitlab_sync-nextcloud-blue?style=for-the-badge&logo=gitlab)](https://gitlab-internal.spirit-dev.net/github-mirror/helm-nextcloud) <!-- markdownlint-disable MD041 -->
[![GitHub Mirror](https://img.shields.io/badge/github_mirror-nextcloud-blue?style=for-the-badge&logo=github)](https://github.com/spirit-dev/helm-nextcloud)
[![App Status](https://argocd-internal.spirit-dev.net/api/badge?name=nextcloud-turingpi&revision=true&showAppName=true)](https://argocd-internal.spirit-dev.net/applications/nextcloud-turingpi)

<!--TOC-->

- [Installation process](#installation-process)
- [Run manual command](#run-manual-command)
- [in case of issues](#in-case-of-issues)
  - [Bad file format](#bad-file-format)
  - [MariaDB recurrent restart](#mariadb-recurrent-restart)
  - [OCC Upgrades](#occ-upgrades)

<!--TOC-->

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

Option 1 - from official doc [github](https://github.com/nextcloud/helm/blob/main/charts/nextcloud/README.md#running-occ-commands)

```bash
# $NEXTCLOUD_POD should be the name of *your* nextcloud pod :)
#> kubectl exec $NEXTCLOUD_POD -- su -s /bin/sh www-data -c "php occ myocccomand"
```

Option 2:

```bash
#> runuser --user www-data -- php occ upgrade/other

Using kubectl:

#> kubectl exec -n [ns] [pod] nextcloud -- runuser --user www-data -- php occ upgrade/other

```

## in case of issues

- Use the web upgrade -> [values.turingpi.yaml#67](helm/values.turingpi.yaml#67)

### Bad file format

> Bad file format reading the append only file appendonly.aof.145.incr.aof: make a backup of your AOF file, then use ./redis-check-aof --fix <filename.manifest>

If the issue concerns the file format.

Please do the following operations:

1. Scale down Redis StatefulSets to 0
2. execute `kshell`
   1. `chown -R 1001:1001 /data`
   2. `apk add --no-cache redis`
   3. `redis-check-aof --fix /data/appendonlydir/appendonly.aof.<number>.incr.aof` > Y > Enter
3. Scale up Redis StatefulSets

### MariaDB recurrent restart

MariaDB is recurrently restarting for no apparent reasons, with this type of log:

```txt
2024-11-18 22:51:20 0 [Note] mysqld: Event Scheduler: Loaded 0 events
2024-11-18 22:51:20 0 [Note] /opt/bitnami/mariadb/sbin/mysqld: ready for connections.
Version: '11.4.4-MariaDB'  socket: '/opt/bitnami/mariadb/tmp/mysql.sock'  port: 3306  Source distribution
/opt/bitnami/mariadb/bin/mysql_upgrade: Deprecated program name. It will be removed in a future release, use '/opt/bitnami/mariadb/bin/mariadb-upgrade' instead
This installation of MariaDB is already upgraded to 11.4.2-MariaDB.
There is no need to run mariadb-upgrade again for 11.4.4-MariaDB.
You can use --force if you still want to run mariadb-upgrade
find: '/docker-entrypoint-startdb.d/': No such file or directory
mariadb 22:51:22.29 INFO  ==> Stopping mariadb
2024-11-18 22:51:22 0 [Note] /opt/bitnami/mariadb/sbin/mysqld (initiated by: unknown): Normal shutdown
2024-11-18 22:51:22 5 [Warning] Aborted connection 5 to db: 'unconnected' user: 'unauthenticated' host: 'localhost' (This connection closed normally without authentication)
2024-11-18 22:51:22 4 [Warning] Aborted connection 4 to db: 'unconnected' user: 'unauthenticated' host: 'localhost' (This connection closed normally without authentication)
2024-11-18 22:51:22 0 [Note] InnoDB: FTS optimize thread exiting.
2024-11-18 22:51:22 0 [Note] InnoDB: Starting shutdown...
2024-11-18 22:51:22 0 [Note] InnoDB: Dumping buffer pool(s) to /bitnami/mariadb/data/ib_buffer_pool
2024-11-18 22:51:22 0 [Note] InnoDB: Buffer pool(s) dump completed at 241118 22:51:22
2024-11-18 22:51:22 0 [Note] InnoDB: Removed temporary tablespace data file: "./ibtmp1"
2024-11-18 22:51:22 0 [Note] InnoDB: Shutdown completed; log sequence number 374572491; transaction id 1094860
2024-11-18 22:51:23 0 [Note] /opt/bitnami/mariadb/sbin/mysqld: Shutdown complete
```

This issue is most likely due to the Liveness and Readiness setup.

In fact, MariaDB is hell of slow to startup. My guess is that the recurring log `Aborted connection 5 to db: [...]` is due a shutdown signal sent by liveness and/or readiness

Higher the `initialDelaySecond` will help

### OCC Upgrades

One thing to mention is that an upgrade might take long, very long to process ! 10 to 15 minutes easy !

To avoid issues while executing upgrades, a nice trick is to disable `liveness|readiness|startup probes` [values.turingpi.yaml#178](helm/values.turingpi.yaml#178)

```diff
  livenessProbe:
-    enabled: true
+    enabled: false
    periodSeconds: 45
  readinessProbe:
-    enabled: true
+    enabled: false
    periodSeconds: 45
  startupProbe:
-    enabled: true
+    enabled: false
    initialDelaySeconds: 120
```
