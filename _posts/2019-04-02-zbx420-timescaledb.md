---
title: "Zabbix 4.2 and Timescaledb"
date: 2019-04-02T16:12:44+02:00
categories: ["favorite-tools", "tools"]
tags: ["Tools", "Favorite Tools", "Zabbix", "Monitoring", "Timeseries", "Timescale", "Open Source"]
readtime: true
---

Today Zabbix released their Version 4.2 into the wild with a great set of new features. There's a good overview on [Zabbix own site](https://www.zabbix.com/whats_new_4_2) but be sure to also check the ["What's new in Zabbix 4.2"](https://www.zabbix.com/documentation/4.2/manual/introduction/whatsnew420) section in the documentation as it's more complete!

One new feature is the experimental support of [TimescaleDB](https://www.timescale.com/). A currently trending Open Source Timeseries Database - well sort of as it's an extension to Postgres.

<!--more-->

---

EDIT on 14.04.2019

Zabbix official docker containers now have an environment switch for enabling TimescaleDB support out-of-the-box!
In .env_db_pgsql is the commented line `# ENABLE_TIMESCALEDB=true`
Just uncomment it and you're good to go!

---


This support for an actual Timeseries DB is way long overdue and setting on TimescaleDB is a good move from Zabbix in my opinion. But as it is currently only experimental the documentation lacks some infos and an out-of-the-box support on installs. So here's a quick way to try it out - that sayed it's experimental and so I wouldn't ingest all my precious data.

As always I will opt for a "Docker install" (with docker-compose and non swarm usage) but won't go through the basics - Zabbix now also official support using containers! Have a [look in the documentation](https://www.zabbix.com/documentation/4.2/manual/installation/containers) and the [GitHub repository](https://github.com/zabbix/zabbix-docker).

After you modified the base files to your wishes, you opt-in Timescales TimescaleDB container. Again I'm refering to [their own install documentation](https://docs.timescale.com/v1.2/getting-started/installation/docker/installation-docker).

```docker
# in zabbix docker-compose file under services
 postgres-server:
  image: timescale/timescaledb:latest-pg11 # <- just change this line!
  volumes:
   - ./zbx_env/var/lib/postgresql/data:/var/lib/postgresql/data:rw
```

All you do here is to replace the default postgres docker image with the TimescaleDB one and leave the rest alone. Next up you have to initialize the DB so let's start the whole stack. But don't rush in and do something in Zabbix for now - maybe just login to check if all is working. If all gone good shutdown all containers and just start the TimescaleDB container alone.

```shell
docker-compose down && docker-compose up -d postgres-server
```

Next up we enable and "migrate" the postgres tables Zabbix has created on first start to TimescaleDB hypertables. Jump into the container and execute the following commands where we enable the extension and migrate the tables.

> !! Always have a look [at the documentation](https://www.zabbix.com/documentation/4.2/manual/appendix/install/timescaledb) first as the actual commands can change each release!
>
> The table migration commands can be extracted from the [zabbix source files](https://www.zabbix.com/download_sources) in "database/postgres/timescale.sql". The migration can take some loooooooooong time for tables with loads of data.

```shell
docker exec -it <postgres-server-containername> psql -U postgres
# then in the container:
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
SELECT create_hypertable('history', 'clock', chunk_time_interval => 86400, migrate_data => true);
SELECT create_hypertable('history_uint', 'clock', chunk_time_interval => 86400, migrate_data => true);
SELECT create_hypertable('history_log', 'clock', chunk_time_interval => 86400, migrate_data => true);
SELECT create_hypertable('history_text', 'clock', chunk_time_interval => 86400, migrate_data => true);
SELECT create_hypertable('history_str', 'clock', chunk_time_interval => 86400, migrate_data => true);
SELECT create_hypertable('trends', 'clock', chunk_time_interval => 86400, migrate_data => true);
SELECT create_hypertable('trends_uint', 'clock', chunk_time_interval => 86400, migrate_data => true);
UPDATE config SET db_extension='timescaledb',hk_history_global=1,hk_trends_global=1;
```

Check the output for errors and if everything works exit the container and start the whole stack again. You should now have Zabbix running atop TimescaleDB and a much more sane stack in my opinion. You will see the benefits the more data is ingested and queried. A big thing is also the housekeeping in Zabbix. Before TimescaleDB the data is removed with many DELETE queries which definitely hurts the overall performance. Now with TimescaleDB's chunked tables the obsolete data is dumped as a whole and the burden on performance is way less.

> !! Beware - it's currently not well documented and [somewhat unclear at least to me](https://www.zabbix.com/forum/zabbix-help/376814-housekeeping-and-timescale-timescaledb) if and how the Zabbix Server internal housekeeping process utilizes the TimescaleDB native dump functions. In each case you should test out the configuration ( Zabbix > Administration > General > Housekeeping ) and the impact on your system.

I haven't had time to migrate my private homemonitor to the new release but I used some spare time to experiment with it on a [katacoda playground](https://www.katacoda.com/learn#playgrounds). I can only advise you to play around with new releases and see the new features in action on a safe harbour without actual impact.