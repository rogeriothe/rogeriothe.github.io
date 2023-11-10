+++
author = "Rogério Lima"
title = "How to dump & restore a PostgreSQL database from a docker container"
date = "2023-11-10"
description = "How to dump & restore a PostgreSQL database from a docker container"
tags = [
    "django",
    "orm",
]
thumbnail = "images/postgres.png"

+++

# How to dump & restore a PostgreSQL database from a docker container

### Artigo original acessado em 10/11/2023:
[How to dump & restore a PostgreSQL database from a docker container]([https://www.caktusgroup.com/blog/2019/01/09/django-bulk-inserts/](https://davejansen.com/how-to-dump-and-restore-a-postgresql-database-from-a-docker-container/))

Dave Jansen
3–4 minutos
Database

Dave Jansen

Nov 25, 2019 — 3 min read
How to dump & restore a PostgreSQL database from a docker container

Sometimes you need to quickly dump and restore a PostgreSQL database, but what's the easiest way to do this when your database is in a Docker container?

How to dump & restore a MariaDB/MySQL database from a docker container

In the same vein as my previous post on dumping and restoring your PostgreSQL database, here are mostly copy/paste-able commands for when you’re using a MariaDB or MySQL database. How to dump &amp; restore a PostgreSQL database from a docker containerSometimes you need to quickly dump and restore a

Dave JansenDave Jansen

How to dump & restore a MongoDB database from a docker container

MongoDB is an interesting tool. Dead simple to create and modify records (or, “documents”) that are basically pure JSON goodness, juxtaposed with at times tools or commands that make you scratch your head in utter confusion and/or disbelief. To help prevent me from having to re-figure this out a

Dave JansenDave Jansen

    2021-04-07: Updated the guide with a more up-to-date method for sending a password along with the commands.

I ran into this just today, and thought I'd share one method that I felt was easy, fast and served my purpose. Depending on why you need to dump/restore a database, this might help for you, too.

This quickie assumes you have nothing directly installed on your development machine, so everything is run straight from and to the Docker PostgreSQL container you're running.
Dump using pg_dump

```bash
docker exec -i pg_container_name /bin/bash -c "PGPASSWORD=pg_password pg_dump --username pg_username database_name" > /desired/path/on/your/machine/dump.sql
```

Restore using psql

Since you are not able to provide a password directly through arguments, we rely on the PGPASSWORD environment variable:

```bash
docker exec -i pg_container_name /bin/bash -c "PGPASSWORD=pg_password psql --username pg_username database_name" < /path/on/your/machine/dump.sql
```

Note: If you are attempting to restore data from a custom format dump, you should instead use pg_restore as I described in my How to set up and use Postgres locally article.

Note: By default PostgreSQL keeps importing even when errors occur. If you would instead prefer to stop the import completely upon error, be sure to add --set ON_ERROR_STOP=on to your above command.
Dump and restore in one command

If you, for example, are moving data from one (e.g. manually created) container to another, you could use pipes to do this in one command, like so:

```bash
docker exec -i pg_old_container_name /bin/bash -c "PGPASSWORD=pg_password pg_dump --username pg_username database_name" | docker exec -i pg_new_container_name /bin/bash -c "PGPASSWORD=pg_password psql --username pg_username database_name"
```

Conclusions

There are certainly other ways to achieve something similar, but this method will work in a pinch. Please keep in mind that you should always ensure your production databases are properly backed up, and ideally automatically so. If you use any of these manual steps as a means to create backups, you're probably doing something not entirely correct. These steps are mostly for moving development data around or pulling (partial) production data locally for debugging, or something along those lines.

That's it, really. I hope this note-to-self quickie will be of some help to you as-well.

Thank you.
